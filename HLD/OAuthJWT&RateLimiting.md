🚀 System Design Refresher: OAuth2, JWT & Rate Limiting (Mid‑Level Engineer View)

## 🔐 OAuth2: Authorization, Not Authentication

- 🎯 **Purpose**: Delegated access without sharing passwords
- 🧩 **Key roles**: Resource Owner, Client, Authorization Server, Resource Server
- 🔁 **Common flows**:
  - ✅ **Authorization Code + PKCE**: SPA / mobile (most secure)
  - ✅ **Client Credentials**: Service‑to‑service (no user context)
  - ✅ **Refresh Tokens**: Long‑lived sessions with rotation
- 🏭 **Real‑world use**:
  - “Sign in with Google/GitHub”
  - Third‑party integrations (e.g., calendar access)
  - Internal microservices auth
- ⚠️ **Key challenges**:
  - Token revocation (JWTs are stateless → need short TTL + rotation / introspection / blocklist)
  - Scope design (least privilege, no `*`)
  - Multi‑tenant `tenant_id` in claims
- 🚫 **When NOT to use OAuth2**:
  - Simple internal M2M where mTLS or API keys suffice
  - Single‑service app with no third‑party needs

> 💡 For **authentication**, use **OIDC** (built on OAuth2).

---

## 🎫 JWT: Self‑Contained Token Format

- 📦 **What it is**: `header.payload.signature` — compact, URL‑safe, self‑contained
- ✅ **Pros**:
  - 🚀 Stateless validation (no DB lookup per request)
  - 🌐 Portable across services/domains
  - 🧾 Rich claims: `user_id`, `roles`, `scope`, `tenant_id`
- ❌ **Cons**:
  - 🔒 Hard to revoke mid‑lifetime
  - 📏 Larger payload than opaque tokens
  - ⚠️ Not encrypted by default (don’t store secrets/PII)
- 🎯 **When to use**:
  - Microservices with stateless APIs
  - SPA/mobile with cross‑domain auth
  - Inter‑service auth in distributed systems
- 🔑 **Best practices**:
  - Use **asymmetric signatures**: `RS256`, `ES256`
  - Short **`exp`** (5–15 min) for access tokens
  - Minimal claims: `sub`, `iss`, `aud`, `exp`, `iat`, `scope`, `tenant_id`
  - Validate **issuer**, **audience**, **expiration**, **signature**
  - Implement **key rotation** with `kid`

---

## 🚦 Rate Limiting: Protect Systems from Abuse

- 🎯 **Purpose**:
  - 🛡️ Prevent DoS/DDoS & brute‑force
  - 💰 Enforce quota / pricing tiers
  - ⚙️ Protect downstream services from overload
  - ⚖️ Ensure fair usage across tenants
- 🧮 **Algorithms**:
  - ⏱️ **Fixed Window**: Simple, but bursts at boundaries
  - 📊 **Sliding Window Log**: Accurate, higher memory
  - 🔄 **Sliding Window Counter**: Good trade‑off
  - 🪣 **Token Bucket**: Controlled bursts
  - 💧 **Leaky Bucket**: Smooths traffic
- 🏭 **Real‑world use**:
  - API gateway limits per IP/user/tenant
  - Login endpoints: 5 attempts/min → block/CAPTCHA
  - Public APIs with tiered limits (free vs pro)
  - Protect expensive ops: search, upload, email
- ⚙️ **Design challenges**:
  - Distributed counting (Redis / dedicated service)
  - Clock skew & consistency
  - Avoiding false positives / over‑throttling
  - Handling bursts gracefully
- 📈 **Observability**:
  - Metrics: allowed vs rejected requests per endpoint/user/tenant
  - Logs with client ID/IP, endpoint, reason
  - Alerts on rejection spikes & repeated hits

---

## 🔗 How They Fit Together in Production

1. 🔐 **Auth**: User logs in via OAuth2/OIDC → gets ID token + JWT access token
2. ✅ **Authorization**: API validates JWT signature & claims; enforces scopes/roles
3. 🚦 **Protection**: API Gateway enforces rate limiting per user/IP/tenant
4. 📊 **Observability**: Logs & metrics for auth failures, token rejections, rate limit hits

This combo gives:

- ✅ Secure, delegated access
- ✅ Stateless, scalable token validation
- ✅ Defense against abuse & overload
- ✅ Visibility into security events

---

## 🧠 System Design Interview: What Interviewers Expect

You must **voluntarily address** auth, abuse protection, and observability in any large system design.

**Key questions to be ready for:**

- 🔐 “How do you handle login & third‑party integrations?”  
  → OAuth2/OIDC, auth code + PKCE, short‑lived tokens + refresh rotation
- 🧾 “Where do you store tokens on the client?”  
  → Discuss XSS/CSRF, BFF pattern, httpOnly cookies vs localStorage
- 🚫 “How do you revoke a compromised token?”  
  → Short TTL, refresh rotation, introspection, Redis blocklist
- 🎫 “JWT vs opaque tokens?”  
  → Statelessness & portability vs revocation difficulty
- 🔑 “How do you handle key rotation?”  
  → Multiple signing keys, `kid`, gradual migration
- 🚦 “How do you protect your API from abuse?”  
  → Rate limiting at gateway + service level, per‑user/IP/tenant
- 🌐 “Distributed rate limiting?”  
  → Redis counters, atomic ops, consistent hashing
- 📊 “What metrics & alerts?”  
  → Allowed/rejected requests, latency impact, abuse patterns

---

## ✅ Mid‑Level Engineer Checklist

**OAuth2:**

- ✅ Use OIDC for auth, OAuth2 for authorization
- ✅ Auth code + PKCE for public clients
- ✅ Short‑lived access tokens + refresh rotation
- ✅ Least‑privilege scopes

**JWT:**

- ✅ Strong algorithms (`RS256`, `ES256`), avoid `none`
- ✅ Validate `iss`, `aud`, `exp`, signature
- ✅ Minimal claims, no sensitive data (or use JWE)
- ✅ Key rotation with `kid`

**Rate Limiting:**

- ✅ Enforce at gateway + critical endpoints
- ✅ Distributed counters (Redis)
- ✅ Choose algorithm based on burst & latency needs
- ✅ Metrics, logs, alerts for anomalies

**Observability:**

- ✅ Log auth failures, token rejections, rate limit hits
- ✅ Track auth latency & rejection reasons
- ✅ Integrate with distributed tracing

---

Designing secure, scalable systems isn’t just about knowing OAuth2/JWT/rate limiting — it’s about **where** they live, **how** they fail, and **what** you observe when things go wrong.

#SystemDesign #Security #OAuth2 #JWT #RateLimiting #Observability #BackendEngineering #Microservices #MAANG #SoftwareEngineering #APIDesign #Auth #Scalability
