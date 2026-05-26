# Video Compression

Video compression is essential to make video transmission, storage, and realtime delivery practical at scale; it uses codecs (algorithms + formats) like H.264, HEVC, and AV1 that trade quality, bitrate, latency, and compute, and following industry standards and best practices is critical for interoperability, cost control, and user experience in large systems.

- **Why it’s necessary**: Raw video is huge (uncompressed frames × frame rate); compression reduces bits dramatically so video fits on networks and disks and plays smoothly for users.

- **How it works (high-level)**: Encoders remove redundancy in space (within a frame) and time (between frames), transform pixels into frequency coefficients (DCT or similar), quantize (lossy step) and entropy-code the result; decoders reverse that to display video.

- **Common terms you’ll see**: codec = coder/decoder implementation, container = MP4/MKV (wraps codec streams), bitrate = bits/sec, GOP = group of pictures (I/P/B frames), keyframe (I-frame) = full frame, P/B = predicted/interpolated frames.

- **Families and tradeoffs**: H.264/AVC is broadly supported and efficient; HEVC/H.265 improves compression ~2x over H.264 at higher complexity; AV1 targets royalty-free licensing with efficiency similar to or better than HEVC but with higher encode cost; newer codecs (VVC, EVC) push efficiency further at more compute cost.

- **Core algorithmic building blocks**: motion estimation/compensation for temporal redundancy, block transforms (DCT-like) for spatial redundancy, quantization for bitrate control (lossy), and entropy coding (CABAC/CAVLC or arithmetic) for final packing.

- **Profiles, levels, and containers**: Profiles/levels constrain features and complexity for devices; containers (MP4, WebM) carry codec streams plus audio and metadata—pick combinations based on target platforms and DRM needs.

---

## How compression helps large-scale applications (system-level impacts)

- **Bandwidth cost reduction**: Better codecs and bitrate ladders reduce egress cost dramatically — a 2x compression gain halves CDN and network costs for the same quality.

- **Storage savings and caching efficiency**: Lower bitrates reduce storage footprint and increase cache hit effectiveness, improving CDN efficiency and cache TTL economics.

- **Reduced startup and rebuffering**: ABR + proper chunking/keyframes reduce startup times and rebuffer events, improving retention and engagement.

- **Scalability of live services**: hardware-assisted encoders and optimized GOPs reduce compute required for real-time encoding (important in live events), enabling more concurrent channels for the same infra.

- **Interoperability and client experience**: following standards ensures content plays across devices and browsers without workaround layers, reducing client complexity and support cost.

---

## Concrete system design pointers

- **Storage model**: ingest mezzanine (high-bitrate, high-quality) -> async transcoding into ABR renditions -> store renditions in object store + push to CDN origin. Keep mezzanine for future re-encodes. Use object lifecycle rules to manage cost.

- **Transcoding pipeline**: orchestrate jobs (Kubernetes + queue), use autoscaling groups of hardware/software encoders, prefer GPU encoders for live and offline bulk transcodes for cost/perf sweet spot.

- **CDN and ABR delivery**: generate HLS/DASH manifests referencing segment URLs aligned to keyframes; enable CDN caching per rendition and use consistent segment lengths (2–6s) for a balance of latency and cacheability.

- **Monitoring and SLOs**: track startup time, rebuffer rate, average bitrate, encoding failures, and cost-per-minute; use these to tune GOP, bitrate ladders, and autoscaling thresholds.

- **Security and DRM**: integrate DRM workflows (Widevine/PlayReady/FairPlay) early if needed; DRM affects container/packaging choices.

---

## Industry standards and licensing (why they matter)

- **Standards (H.26x, MPEG, AV1/AOM)**: provide consistent bitstream syntax, decoder expectations, and broad ecosystem support; using standards simplifies cross-device playback and gives access to optimized hardware decoders.

- **Patent/licensing realities**: some efficient codecs (HEVC) are patent-encumbered with licensing fees; AV1/VP9 were developed to be more royalty-friendly but have higher encode costs—these business constraints influence codec choice for scale.

- **Practical rule**: target the highest-efficiency codec that your audience’s devices support in hardware, and provide fallbacks (H.264/MP4) to maximize compatibility while minimizing licensing/compute cost.

---

## Quick example (practical choices)

- **For web + mobile audience**: encode primary ABR ladder in H.264 (broad compatibility) and offer HEVC/AV1 for devices/browsers that support hardware decode; store mezzanine in DNxHR/ProRes or lossless format for future proofing.

- **Segment length**: 4s segments, GOP aligned to segments, keyframe every 2–4 seconds for low-latency VOD; for low-latency live consider CMAF and chunked CMAF with sub-second segments.
