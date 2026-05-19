# System Design Refresher: Security & Observability

**Topic:** OAuth2, JWT, Rate Limiting — Real‑world implications, trade‑offs, and interview angles

## 1. Core Concepts (Foundation)

### 1.1 OAuth2 — Authorization Framework, Not Authentication

**What it is:**  
OAuth2 is a protocol that lets a third‑party app get **limited access** to a user’s resources **without sharing the user’s password**. It defines _flows_ (grant types) for obtaining an **access token**.

**Key roles:**

- **Resource Owner**: the user
- **Client**: the app requesting access
- **Authorization Server**: issues tokens (e.g., `auth.google.com`)
- **Resource Server**: hosts protected APIs (e.g., `api.google.com`)

**Common flows (grant types):**
| Flow | When to use | Security notes |
|------|-------------|----------------|
| Authorization Code | Server‑side apps, SPAs with PKCE | Most secure; token not exposed in URL |
| PKCE (with Auth Code) | Mobile & SPA apps | Prevents interception attacks
| Client Credentials | Machine‑to‑machine (service→service) | No user context; backend only |
| Refresh Token | Long‑lived sessions | Must be stored securely; rotated |

**Real‑world use cases:**

- “Sign in with Google/Facebook/GitHub”
- Third‑party integrations (e.g., a calendar app accessing your Google Calendar)
- Internal microservices: service A gets an access token from an internal OAuth2 server to call service B

**Why we use it:**

- Decouples authentication from authorization
- Enables fine‑grained scopes (`read:email`, `write:drive`)
- Avoids hard‑coding credentials in clients

---

### 1.2 JWT (JSON Web Token) — Self‑Contained Token Format

**What it is:**  
JWT is a **token format**, not a protocol. It’s a compact, URL‑safe string with three parts:  
`header.payload.signature`

- **header**: algorithm & token type (e.g., `{"alg":"RS256","typ":"JWT"}`)
- **payload**: claims (e.g., `sub`, `iss`, `exp`, `scope`, `user_id`)
- **signature**: cryptographically signed by the issuer’s private key

**Why JWTs are useful:**

- **Stateless validation**: server verifies signature without DB lookup
- **Portable**: can be used across services/domains
- **Rich claims**: can embed user ID, roles, scopes, expiration

**Common misuse:**

- Storing sensitive data (JWT is **not encrypted** by default; anyone can decode the payload)
- Using weak algorithms (`none`, `HS256` with short secrets)
- No expiration or very long `exp` → high impact if stolen

**Real‑world use cases:**

- Access token in OAuth2 flows (often a JWT)
- Session token for SPA ↔ API authentication
- Inter‑service authentication in microservices (service A presents JWT to service B)

---

### 1.3 Rate Limiting — Protecting Systems from Abuse

**What it is:**  
Rate limiting restricts how many requests a client can make in a given time window.

**Why we use it:**

- Prevent **DoS / DDoS** and brute‑force attacks
- Enforce **quota / pricing tiers** (free vs paid)
- Protect **downstream services** from overload
- Ensure fair usage across tenants

**Common algorithms:**
| Algorithm | How it works | Pros / Cons |
|-----------|--------------|-------------|
| Fixed Window | Count requests in fixed intervals (e.g., 100/min) | Simple; can allow bursts at window boundaries |
| Sliding Window Log | Track exact timestamps of each request | Accurate; more memory |
| Sliding Window Counter | Hybrid of fixed + sliding | Good trade‑off |
| Token Bucket | Tokens added at rate; each request consumes one | Allows controlled bursts |
| Leaky Bucket | Requests leak out at constant rate | Smooths traffic; no bursts |

**Real‑world use cases:**

- API gateways limiting per‑IP or per‑user (e.g., 1000 req/hour)
- Login endpoints: 5 attempts/min → block or CAPTCHA
- Public APIs with tiered limits (free: 100/day, pro: 10k/day)
- Protecting expensive operations (search, image upload, email sending)

---

## 2. Design, Trade‑offs, and Pitfalls

At this level, I expect you to:

- Choose the right mechanism for the problem
- Understand **where** in the architecture it lives
- Discuss **failure modes**, **scalability**, and **operational concerns**
- Articulate trade‑offs clearly in design interviews

---

### 2.1 OAuth2 in Production

#### Where it fits in the architecture

Typical flow for a web app with an API:

1. Client (browser) redirects user to **Authorization Server**.
2. User authenticates; Authorization Server returns **authorization code**.
3. Backend exchanges code for **access token** (+ optional **refresh token**).
4. Backend calls **Resource Server** APIs with `Authorization: Bearer <access_token>`.

**Key design decisions:**

- **Who holds the tokens?**
  - SPA: tokens in memory (not localStorage if XSS risk is high)
  - Backend‑for‑Frontend (BFF): tokens stored server‑side, cookie‑based session to frontend
- **Public vs confidential clients:**
  - Mobile/SPA = public → must use PKCE
  - Server‑side = confidential → can use client secret
- **Token lifetime:**
  - Short‑lived access tokens (5–15 min)
  - Longer‑lived refresh tokens with rotation & revocation

#### Implementation challenges

- **Token revocation**:  
  JWTs are stateless; once issued, they’re valid until expiry unless you add:
  - Token introspection endpoint (adds latency)
  - Short TTL + refresh token rotation
  - Blocklist in cache (Redis) with TTL
- **Scope design**:  
  Over‑broad scopes (`*`) are dangerous. Design least‑privilege scopes per use case.
- **Multi‑tenant & multi‑org**:  
  Need to encode `org_id` / `tenant_id` in claims and enforce on every request.
- **Migration & versioning**:  
  Supporting multiple grant types & legacy clients while moving to safer flows.

#### When **not** to use OAuth2

- Simple internal service‑to‑service where mutual TLS or API keys are enough
- Single‑service app where you control both client and server and don’t need third‑party access
- When you just need basic login: consider **OIDC** (built on OAuth2) or simpler session‑based auth

> Note: For authentication (who are you?), **OIDC** is the standard layer on top of OAuth2. OAuth2 alone is authorization.

---

### 2.2 JWT in Production

#### When to use JWT

Good fits:

- Stateless APIs with many microservices
- Cross‑domain / cross‑service authorization
- Mobile/SPA where you want to avoid server‑side session stores

Bad fits:

- When you need **immediate revocation** and can’t tolerate short TTL
- When you must store **sensitive data** in the token
- When you can’t securely manage signing keys (rotation, HSM, KMS)

#### Key trade‑offs

| Aspect        | JWT pros                              | JWT cons                               |
| ------------- | ------------------------------------- | -------------------------------------- |
| Statelessness | No DB lookup per request              | Hard to revoke mid‑lifetime            |
| Performance   | Fast signature verify                 | Larger payload than opaque token       |
| Portability   | Works across services/domains         | Must protect against theft (XSS, MITM) |
| Observability | Claims visible in logs (if sanitized) | Can leak info if not careful           |

#### Best practices

- Use **asymmetric signatures** (`RS256`, `ES256`) for public clients; symmetric (`HS256`) only for internal, tightly controlled systems.
- Set **short `exp`** (5–15 min) for access tokens; use refresh tokens for longevity.
- Include **minimal claims**: `sub`, `iss`, `aud`, `exp`, `iat`, `scope`, `tenant_id`.
- **Never** store secrets, passwords, or PII unless encrypted (JWE).
- Implement **key rotation** and have a way to invalidate old keys.
- Validate:
  - `iss` (issuer)
  - `aud` (audience)
  - `exp`, `nbf` (time claims)
  - signature algorithm

---

### 2.3 Rate Limiting in Production

#### Where to implement

- **API Gateway** (Kong, Envoy, AWS API Gateway, Cloudflare): central policy, per‑IP / per‑user / per‑API
- **Service level**: per‑endpoint rate limits for expensive operations
- **Per‑tenant / per‑org**: for SaaS multi‑tenancy

#### Design challenges

- **Distributed counting**:  
  In a scaled‑out system, you need a shared counter (Redis, Memcached, dedicated rate limit service).
- **Clock skew & consistency**:  
  Sliding window algorithms need synchronized clocks or careful design.
- **False positives**:  
  Overly strict limits cause legitimate traffic to be blocked; need adaptive thresholds & monitoring.
- **Burst handling**:  
  Users expect some burst; token bucket or sliding window with burst allowance is better than strict fixed window.

#### Trade‑offs

| Approach                      | Complexity | Accuracy | Burst support     |
| ----------------------------- | ---------- | -------- | ----------------- |
| Fixed window (simple counter) | Low        | Medium   | Poor              |
| Sliding window log            | High       | High     | Good              |
| Token bucket                  | Medium     | High     | Good (controlled) |
| Gateway‑only                  | Low        | Medium   | Depends on config |

#### Observability needs

- Expose metrics:
  - Requests allowed vs rejected per endpoint / user / tenant
  - Latency impact of rate limit checks
- Log rate limit rejections with:
  - Client ID / IP
  - Endpoint
  - Reason (quota exceeded, global limit, etc.)
- Alert when:
  - Rejection rate spikes unexpectedly
  - A single client hits limits repeatedly (possible abuse or bug)

---

## 3. How These Pieces Fit Together

Typical secure API setup:

1. **Authentication**:
   - User logs in via OAuth2 + OIDC → gets ID token + access token (JWT).
2. **Authorization**:
   - API validates JWT signature & claims; checks scopes/roles.
3. **Protection**:
   - API Gateway enforces rate limiting per user/IP/tenant.
4. **Observability**:
   - Logs & metrics for auth failures, token rejections, rate limit hits.
   - Tracing across auth service, API gateway, and backend services.

This combination gives you:

- Secure, delegated access (OAuth2/OIDC)
- Stateless, scalable token validation (JWT)
- Defense against abuse & overload (rate limiting)
- Visibility into security events (observability)

---

## 4. System Design Interview Angle: What Interviewers Look For

In a system design interview, you’re expected to show **high‑level understanding** plus **pragmatic trade‑off thinking**.

### 4.1 Typical Interview Prompts

You might see prompts like:

- “Design Twitter’s API”
- “Design a URL shortener”
- “Design a ride‑sharing service”
- “Design a payment API”

In all of these, the interviewer expects you to **voluntarily address**:

- How users authenticate and authorize
- How you protect APIs from abuse
- How you observe security‑related issues

### 4.2 Key Questions You Should Be Ready to Answer

**OAuth2 & Authentication/Authorization:**

1. “How would you handle user login and third‑party integrations?”
   - Expect: OAuth2/OIDC flow, authorization code + PKCE for SPA/mobile, short‑lived access tokens + refresh tokens.
2. “Where do you store tokens on the client?”
   - Expect: discussion of XSS/CSRF, BFF pattern, httpOnly cookies vs localStorage.
3. “How do you revoke a token if a user’s session is compromised?”
   - Expect: short TTL, refresh token rotation, token introspection, or blocklist in Redis.

**JWT:** 4. “Why use JWT over opaque tokens?”

- Expect: statelessness, cross‑service portability, vs revocation difficulty and size.

5. “What claims would you include and why?”
   - Expect: `sub`, `iss`, `aud`, `exp`, `scope`, `tenant_id`; minimal PII.
6. “How do you handle key rotation?”
   - Expect: multiple signing keys, versioned key IDs (`kid`), gradual migration.

**Rate Limiting:** 7. “How would you protect your API from abuse?”

- Expect: rate limiting at gateway + service level, per‑user/per‑IP/per‑tenant, algorithms (token bucket, sliding window).

8. “How do you implement rate limiting in a distributed system?”
   - Expect: shared counter in Redis, atomic operations, consistent hashing, fallback strategies.
9. “What metrics and alerts would you set up?”
   - Expect: allowed/rejected requests, latency impact, abuse patterns, sudden spikes.

**Trade‑off & Observability:** 10. “What are the trade‑offs of using JWT in a high‑security system?”

- Expect: stateless scalability vs revocation difficulty; mitigations via short TTL, rotation, introspection. 11. “How do you balance security and user experience with rate limiting?”
- Expect: tiered limits, progressive throttling, CAPTCHA/Math challenge, soft limits + alerts before hard blocks. 12. “How would you detect and respond to a security incident (e.g., token theft)?”
- Expect: logs, anomalies in auth failures, rate limit spikes, automated revocation, forced re‑login, alerting on‑call.

---

## 5. Best Practices Summary (what I expect you to internalize)

**OAuth2:**

- Use **OIDC** for authentication; OAuth2 for authorization.
- Prefer **authorization code + PKCE** for public clients.
- Keep **access tokens short‑lived**, use **refresh token rotation**.
- Design **least‑privilege scopes**.

**JWT:**

- Treat JWT as **opaque to the user**; don’t trust client‑side logic.
- Use **strong algorithms** (`RS256`, `ES256`); avoid `none`.
- Validate **issuer, audience, expiration, signature**.
- Avoid storing sensitive data; use **JWE** if encryption is needed.

**Rate Limiting:**

- Enforce at **API gateway** + critical service endpoints.
- Use **distributed counters** (Redis) for consistency.
- Choose algorithm based on **burst needs** and **latency constraints**.
- Expose **metrics & logs**; set alerts for anomalies.

**Observability:**

- Log auth failures, token rejections, rate limit hits with context.
- Track:
  - Auth latency
  - Token validation failures by reason
  - Rate limit rejection rates per tenant/user/endpoint
- Integrate with **tracing** to see full request path through auth & gateway.
