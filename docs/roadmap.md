# nano-rag Development Roadmap

本ドキュメントは、プロジェクトのMVP完成およびOSS公開までの12週間（約3ヶ月）の開発ロードマップ、評価プロトコル、およびリスク管理を定義する。

## 1. 開発フェーズと週単位タスク分解 (12-Week Plan)

### Phase 1: Core Engine Development (Week 1 - 4)
RustによるONNX推論とHNSWインデックスエンジンの構築。

*   **Week 1**: プロジェクトセットアップ。Rustライブラリ初期化、ONNX Runtime Mobileコンパイル検証。
*   **Week 2**: Sentence-Transformersモデルのint8量子化、推論パイプライン構築。
    *   *DoD (Definition of Done)*: テキストから384次元int8ベクトルが生成され、精度劣化がベースラインの95%以上を維持すること。
*   **Week 3**: HNSWアルゴリズムのRust実装（mmapバックエンド対応）。
    *   *DoD*: ベクトル1万件の挿入、検索が正しく機能し、ユニットテストのカバレッジ80%以上。
*   **Week 4**: SQLite/Zstdを用いたドキュメントストアとHNSWの結合、Arena Allocator実装。
    *   *DoD*: テキスト入力→ベクトル化→保存→クエリの統合テストがパスすること。

### Phase 2: Android Integration & Optimization (Week 5 - 8)
JNIバインディングとAndroid特有の電力・メモリ最適化。

*   **Week 5**: JNIバインディング実装、`DirectByteBuffer`を用いたZero-Copyインターフェースの構築。
    *   *DoD*: Kotlin側からGCをトリガーせずにインサートとクエリが実行できること。
*   **Week 6**: KotlinラッパーAPI設計、Androidライブラリ（AAR）化のビルドパイプライン（Gradle + Cargo）構築。
*   **Week 7**: WorkManager連携による遅延インデックス構築、サーマルスロットリング制御（Thermal API連携）の実装。
    *   *DoD*: バッテリー充電中・アイドル時のみバックグラウンドでインデックス処理が走ることを確認。
*   **Week 8**: NNAPI / XNNPACK の動的ルーティング実装。
    *   *DoD*: Tensor搭載機(NNAPI)と非搭載機(XNNPACK)で正常に推論が実行され、クラッシュしないこと。

### Phase 3: Benchmarking, Packaging & OSS Launch (Week 9 - 12)
ベンチマーク取得、ドキュメント化、およびパブリックリリース。

*   **Week 9**: ベンチマークデータセットによる精度・パフォーマンステスト実施。
*   **Week 10**: Android Battery HistorianとPerfettoを用いた電力・メモリプロファイリング、ボトルネック解消。
    *   *DoD*: Architecture.mdで定義したKPI（ピーク電力<1W, RAM<50MB, クエリ<50ms）を全項目クリア。
*   **Week 11**: サンプルアプリ実装（Gemini Nano / Android AICore連携のデモ）、APIドキュメント（Dokka/Rustdoc）作成。
*   **Week 12**: Maven Central公開、GitHubリポジトリのOSS化、Medium/Zennでの技術ブログ公開。

## 2. ベンチマーク用データセットと評価プロトコル

**データセット**:
1. **Wikipedia Subsets (SQuAD v2 ベース)**: 汎用的な知識検索のレイテンシと精度（Recall@5）の測定。
2. **Simulated Personal Data**: LINE/Slackのチャットエクスポートデータ、標準メモアプリの形式を模したダミーデータ（10,000チャンク）。文脈のチャンキング精度の評価に使用。

**評価プロトコル**:
- **Macrobenchmark**: AndroidX Macrobenchmarkを利用し、実機でのコールドスタートおよびクエリレイテンシ（p50, p90, p95）を自動測定。
- **Battery Historian**: 1万件のインデックス構築タスクをバックグラウンドで実行し、CPU WakeLock、無線LAN使用状況を含めたトータルのmAh消費量を測定。
- **Perfetto Trace**: JNI境界でのシステムコールのオーバーヘッド、およびメモリ割り当てのスパイクを可視化。

## 3. 想定リスクと回避策

| リスク | 影響度 | 回避策 |
| :--- | :--- | :--- |
| **Androidのバージョン断片化** | 高 | サポート対象を **Min SDK 26 (Android 8.0)** に限定。NNAPIの挙動差異を吸収するため、OSバージョンごとのフォールバックロジックを実装。 |
| **端末の性能差（SoCごとの特性）** | 中 | 端末プロファイル（High/Mid/Low）をアプリ起動時に判定。Lowエンド機ではONNXのバッチサイズを最小化し、OOMを回避する。 |
| **電力プロファイラの測定差異** | 高 | 特定端末での過剰最適化を防ぐため、Pixel 6, 7, 8 および Galaxy S22（Snapdragon環境）の物理デバイスファームをFirebase Test Lab等と併用して確保。 |

## 4. 技術的不確実性への対応

**最大の不確実性**: ONNX Runtime MobileにおけるNNAPIデリゲートの安定性。
特定のAndroidベンダー（特にカスタムROMや安価なMediaTek SoC）では、NNAPIのドライバにバグがあり、推論結果がNaNになる、またはクラッシュする事例が報告されている。

**対応策（Fallback Strategy）**:
SDK初期化時に、テスト用ダミーテンソルを用いた「ウォームアップ・バリデーション」を実行する。
推論結果のベクトルが期待値（事前にハードコードしたハッシュ値）と大きく乖離する場合、あるいはタイムアウトが発生した場合は、そのセッションではNNAPIを無効化し、強制的にCPU（XNNPACK）モードへフォールバックするフェイルセーフ機構を実装する。これによりSDKの致命的クラッシュ率（Crash-Free Users）99.9%以上を担保する。
