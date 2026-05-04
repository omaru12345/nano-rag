# 02 — Scope

A small team can only win by being narrower than the competition. This file defines
the hard scope. Anything outside it must be rejected, even if technically interesting.

## DO focus on

- RAM footprint (≤ 50 MB peak) and battery cost (≤ 1 W during indexing)
- Query latency (≤ 50 ms p95 for top-k)
- Zero-copy JNI boundary, mmap-backed storage
- Privacy: nothing leaves the device, ever
- A drop-in Kotlin API that an Android app can integrate in ≤ 3 lines

## DO NOT build

- **Your own LLM.** Gemini Nano (and equivalents) are the consumer; we feed them context.
- **iOS support before Phase 4.** Android first. Cross-platform is a distraction now.
- **Cloud sync / multi-device features.** On-device is the entire pitch.
- **A general-purpose vector database for servers.** That market is crowded and irrelevant
  to the acquisition target. Stay mobile-only.
- **Custom embedding model training.** Use existing small Sentence-Transformers; the moat
  is the runtime, not the model.

## Why this matters

Every "DO NOT" item above has been considered and rejected because it would either
(a) duplicate Google's existing capability, (b) dilute the technical moat (on-device,
mobile, low-power), or (c) push the team toward operational work that a 1–5 person
crew cannot sustain.

If you believe a "DO NOT" item should be reopened, escalate to the human owner —
do not start implementing it.
