# Adaptive Streaming: Transcoding, Bitrate, and Resolutions

Adaptive streaming works by transcoding one source video into multiple renditions (different bitrates + resolutions), segmenting them into 2–10s chunks, and letting the player dynamically switch between renditions based on real-time bandwidth.

## Core concepts

### Bitrate

- `Definition`: data rate of the stream, measured in kbps or Mbps (kilobits/megabits per second).

- `Relationship`: higher bitrate → higher quality, but more data used.

- `Not bandwidth`: bandwidth is the maximum capacity; bitrate is the actual data rate being sent.

### Resolution

- Pixel dimensions (e.g., 640×480, 1280×720, 1920×1080).

- Higher resolution needs higher bitrate to look good, but the exact bitrate depends on codec and content complexity.

### Encoding ladder (key concept)

A predefined set of bitrate–resolution pairs your platform supports. Example (Netflix-style ladder):

| Bitrate (kbps) | Resolution |
| -------------- | ---------- |
| 235            | 320×240    |
| 560            | 512×384    |
| 1050           | 640×480    |
| 2350           | 1280×720   |
| 4300           | 1920×1080  |
| 5800           | 1920×1080  |

The player chooses the highest rung it can sustain given current network conditions.

### Transcoding vs encoding

- `Encoding`: raw camera feed → compressed video (first time compression).

- `Transcoding`: already compressed video (mezzanine) → multiple compressed renditions at different bitrates/resolutions.

- For ABR, we transcode one master file into many renditions.

### Segmenting

- Video is split into small chunks (2–10 seconds).

- **Allows the player** to switch bitrate mid-stream without reloading the entire file.

### Manifest file

- A playlist that lists all renditions and segment URLs:
  - `HLS`: .m3u8 playlist

  - `MPEG-DASH`: .mpd (Media Presentation Description).

- Player reads the manifest, then requests segments at the chosen bitrate.

---

## How adaptive streaming works end-to-end

- **Ingest**: Upload or live ingest of a master file / stream.

- **Transcode**: Create multiple renditions (different bitrate + resolution).

- **Segment**: Split each rendition into short chunks.

- **Package**: Create manifest (HLS/DASH) listing all renditions and segments.

- **Distribute**: Serve via CDN to edge servers.

- **Play**:
  - Player downloads manifest.

  - Starts at a low bitrate.

  - Monitors download speed and buffer level.

  - After each segment, picks the next segment’s `bitrate/resolution` based on current conditions.

**The player** is the decision-maker, not the server.

---

## Real-life example: watching a Netflix show on a phone

Imagine you’re on a train:

- Start: Phone detects weak cellular signal. Player picks `480p` at `~1 Mbps` from the ladder.

- Train enters tunnel, signal drops: Next segment is fetched at `360p / 600 kbps` to avoid buffering.

- Train exits tunnel, signal improves: Player switches to `720p / 2.5 Mbps`, then later `1080p / 4–5 Mbps` if bandwidth allows.

- You don’t manually change quality; the player adapts automatically every few seconds.

This is Adaptive Bitrate Streaming (ABR) in action.

---

## Typical bitrate ranges (H.264, practical)

These are industry-typical, not absolute rules:

| Resolution | Typical bitrate (H.264) |
| ---------- | ----------------------- |
| 360p       | 400–800 kbps            |
| 480p       | 700–1,200 kbps          |
| 720p       | 1,500–3,000 kbps        |
| 1080p      | 3,000–6,000 kbps        |
| 1440p (2K) | 6,000–10,000 kbps       |
| 2160p (4K) | 15,000–25,000 kbps      |

Newer codecs (H.265, VP9, AV1) can achieve similar quality at ~30–50% lower bitrate.

key points:

- You don’t compute bitrate from resolution on the fly.

- You design and maintain an encoding ladder (bitrate–resolution pairs) and tune it per content type (sports vs. talking head) using per-title encoding.

---

## Backend Steps to Support Adaptive Streaming

- **Ingest** → raw mezzanine stored.

- **Orchestrate** → message queue triggers parallel transcoding jobs per rendition.

- **Transcode** → produce video and audio elementary streams per rendition, using the chosen ladder and codec.

- **Package** → segment and wrap into HLS/DASH (CMAF for unified), generate manifest.

- **Protect** (optional) → encrypt with DRM if needed.

- **Distribute** → upload to CDN origin, serve playlist URL.

- **Monitor** → track pipeline health and quality metrics.

- Example FFmpeg sketch (conceptual):

```bash
ffmpeg -i input.mp4 \
  -map 0 -c:v libx264 -b:v 4300k -s 1920x1080 -g 48 \
  -map 0 -c:v libx264 -b:v 2350k -s 1280x720 -g 48 \
  -map 0 -c:v libx264 -b:v 1050k -s 640x480 -g 48 \
  -f hls -master_pl_name master.m3u8 master.m3u8
```

---

## Working Mental Model for Backend Engineers

- Think of ABR as:

“One video → many versions → player picks the right one.”

- Key terms:

Bitrate, resolution, segments, manifest, HLS/DASH, encoding ladder.

- You’re responsible for:
  - Designing the encoding ladder (bitrate–resolution pairs) for your product.

  - Building/scaling the transcoding pipeline (jobs, retries, failures).

  - Ensuring low-latency, reliable delivery via CDN.

  - Integrating and configuring ABR players.

  - Using observability to tune quality-of-experience metrics.

  - Making tradeoffs:
    - More rungs → smoother adaptation but higher storage/compute.

    - Newer codecs → lower bitrate but higher encode cost and compatibility concerns.
