# nano-rag — Development Roadmap

This document covers the 12-week (≈ 3-month) plan to MVP and OSS release, plus the evaluation protocol and risk management.

## 1. Phases and weekly task breakdown (12-week plan)

### Phase 1: Core engine development (Weeks 1–4)
Build the ONNX inference + HNSW index engine in Rust.

*   **Week 1**: Project setup. Initialize the Rust library; verify ONNX Runtime Mobile compilation.
*   **Week 2**: INT8 quantization of a Sentence-Transformers model; assemble the inference pipeline.
    *   *DoD*: Text → 384-dim int8 vector with ≥ 95% of baseline accuracy retained.
*   **Week 3**: Rust HNSW implementation (with mmap backend).
    *   *DoD*: Insert / search 10k vectors correctly. Unit-test coverage ≥ 80%.
*   **Week 4**: Wire the document store (SQLite / Zstd) to HNSW; implement the Arena allocator.
    *   *DoD*: End-to-end integration test (text → vector → store → query) passes.

### Phase 2: Android integration & optimization (Weeks 5–8)
JNI bindings and Android-specific power / memory optimization.

*   **Week 5**: JNI bindings; build the zero-copy interface using `DirectByteBuffer`.
    *   *DoD*: Insert and query from Kotlin without triggering GC.
*   **Week 6**: Design the Kotlin wrapper API. Build the AAR pipeline (Gradle + Cargo).
*   **Week 7**: Deferred indexing via WorkManager. Thermal-throttling control via the Thermal API.
    *   *DoD*: Background indexing only runs while charging / idle.
*   **Week 8**: Adaptive NNAPI / XNNPACK routing.
    *   *DoD*: Inference runs cleanly without crashes on both Tensor (NNAPI) and non-Tensor (XNNPACK) devices.

### Phase 3: Benchmarking, packaging, and OSS launch (Weeks 9–12)
Benchmark numbers, documentation, public release.

*   **Week 9**: Run accuracy / performance tests on the benchmark dataset.
*   **Week 10**: Power and memory profiling with Android Battery Historian and Perfetto; close the bottlenecks.
    *   *DoD*: All KPIs in `architecture.md` met (peak power < 1 W, RAM < 50 MB, query < 50 ms).
*   **Week 11**: Sample app (Gemini Nano / Android AICore integration demo). API docs (Dokka / Rustdoc).
*   **Week 12**: Maven Central release. Open the GitHub repo. Publish technical blog posts (Medium / Zenn).

## 2. Benchmark datasets and evaluation protocol

**Datasets**:
1. **Wikipedia subsets (SQuAD v2 base)**: Generic-knowledge retrieval — measure latency and accuracy (Recall@5).
2. **Simulated personal data**: 10,000 chunks of dummy data shaped like LINE / Slack chat exports and standard notes-app entries. Used to evaluate context chunking quality.

**Evaluation protocol**:
- **Macrobenchmark**: AndroidX Macrobenchmark on real devices, automated cold-start and query latency (p50, p90, p95).
- **Battery Historian**: Total mAh consumption for indexing 10k items in the background (CPU WakeLock, Wi-Fi included).
- **Perfetto Trace**: Visualize JNI-boundary syscall overhead and any allocation spikes.

## 3. Risks and mitigations

| Risk | Severity | Mitigation |
| :--- | :--- | :--- |
| **Android version fragmentation** | High | Cap supported minimum SDK at **26 (Android 8.0)**. Implement OS-version branches to absorb NNAPI behavior differences. |
| **Device performance variance (per-SoC)** | Med | Profile devices into High / Mid / Low at startup. Use the smallest ONNX batch on Low-tier devices to avoid OOM. |
| **Inconsistent power-profiler measurements** | High | To avoid over-fitting to one device, secure physical farms covering Pixel 6, 7, 8 and Galaxy S22 (Snapdragon), supplemented by Firebase Test Lab. |

## 4. Handling technical uncertainty

**Biggest uncertainty**: stability of the NNAPI delegate inside ONNX Runtime Mobile.
On certain Android vendors (especially custom ROMs and some inexpensive MediaTek SoCs), NNAPI driver bugs can cause inference to return NaN or to crash.

**Fallback strategy**:
On SDK init we run a "warm-up validation" with a known dummy tensor. If the embedding deviates significantly from the hard-coded expected hash, or if it times out, we disable NNAPI for the session and force the CPU path (XNNPACK). This fail-safe keeps Crash-Free Users at 99.9%+.
