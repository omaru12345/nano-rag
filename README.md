# nano-rag

A **tiny, low-power RAG SDK** that vectorizes on-device documents (notes, messages, PDFs) locally and feeds context to on-device LLMs (Gemini Nano and similar).

---

## 🎯 Goal: Acquisition by Google

This project targets a **Jetpac-style acqui-hire by Google** (3–5 person team / technical core).

**The acquisition target is the SDK itself.** We polish it to a quality bar where the Android / Pixel team can bundle it directly into the OS.

| Likely buyer | Integration scenario |
|---|---|
| **Android AI Core** | The standard RAG layer that supplies context to Gemini Nano (exposed as an OS-level API). |
| **Pixel team** | Foundation for Pixel's "magical" contextual assistant features. |
| **Workspace mobile** | Offline search across Drive / Gmail / Docs. |

### Technical moat we are paid for
1. **Power consumption** (≤ 5% battery per hour during continuous indexing)
2. **Index size** (10,000 documents in ≤ 50 MB)
3. **Query latency** (top-10 search in ≤ 50 ms)
4. **Memory efficiency** (peak RAM ≤ 100 MB at runtime)

---

## Tech stack

| Layer | Tech |
|---|---|
| Core | Rust (built as a `cdylib` for Android NDK / iOS) |
| Embeddings | A small Sentence-Transformers model exported to ONNX and quantized (< 30 MB) |
| Vector search | HNSW reimplemented in Rust (or a fork of `instant-distance`) |
| Android wrapper | Kotlin + JNI |
| iOS wrapper (later) | Swift + a C interface |

---

## Repository layout

```
nano-rag/
├── core/                # Rust implementation (embeddings + HNSW + storage)
├── android/             # AAR library (Kotlin + JNI)
├── bench/               # Power / latency benchmarks
└── docs/                # API documentation / acquisition pitch
```

---

## Three-phase plan

### Phase 1 (≤ 2 months): Core implementation
- [ ] Embedding computation in Rust (via ONNX Runtime)
- [ ] Working HNSW vector search
- [ ] End-to-end demo CLI on 1,000 documents

### Phase 2 (≤ 4 months): Android SDK
- [ ] Buildable AAR locally
- [ ] Sample Android app (notes app + RAG search)
- [ ] Publish power / latency benchmark numbers

### Phase 3 (≤ 6 months): Visibility
- [ ] Open-source on GitHub (MIT / Apache 2.0)
- [ ] Publish the SDK to Maven Central
- [ ] Get visibility around Android Dev Summit / Google I/O

---

## Development commands

```bash
# Rust core
cd core && cargo build --release && cargo test

# Android SDK build
cd android && ./gradlew assembleRelease

# Benchmarks
cd bench && cargo run --release -- --dataset wiki1k
```

(Implementations land when Phase 1 starts.)
