# Acquisition Thesis: Why Google Acquires nano-rag

本ドキュメントは、Google（特にAndroid AI Core / Pixel / Workspaceモバイルチーム）による「nano-rag」プロジェクトおよび開発チームのAcqui-hire（人材獲得目的の買収）を成立させるための戦略的根拠を定義する。

## 1. 戦略的背景とAndroid AI Coreへの接続

Googleは「Gemini Nano」を通じてオンデバイスAIの覇権を握ろうとしている。しかし、現状のGemini Nanoはコンテキストウィンドウが限られており、ユーザーのローカルデータ（メモ、メール、チャット履歴）をプロンプトに動的に組み込む「オンデバイスRAG」のインフラがAndroidエコシステムにおいて欠如している。

**nano-ragの提供価値**:
nano-ragは、Android AI Coreに対する完璧な「プラットフォーム・プラグイン」として機能する。nano-ragが担うローカルベクターストレージと検索エンジンがAndroid OSに組み込まれれば、すべてのAndroidアプリが標準API経由でローカルRAGを実装可能になる。

## 2. Pixel独自機能（Apple Intelligence対抗）としての価値

Appleは「Apple Intelligence」において、Semantic Indexを用いたシステム全体を横断する文脈理解（Personal Context）をアピールしている。
GoogleがPixel端末でこれに対抗・凌駕するためには、スクリーン上の情報やローカルストレージ内の全テキストをバックグラウンドで省電力にベクトル化し続ける技術が不可欠である。

nano-ragが持つ「1W未満の省電力インデックス構築」「サーマルバジェット管理」の技術は、Pixelの「Personal Context」機能（仮称：Pixel Remember）を実現するためのコア技術として直結する。

## 3. 競合優位性と「技術の堀」

既存のモバイルデータベースと比較し、圧倒的な優位性を持つ。

| 競合製品 | 弱点 | nano-ragの優位性 (技術の堀) |
| :--- | :--- | :--- |
| **ObjectBox** | C++ベース。汎用DBの拡張であり、ベクトル検索専用の最適化（int8量子化など）が弱くメモリを食う。 | **Rustのメモリ安全性**と、mmap/Zero-Copyによる圧倒的な省メモリ（< 50MB）。 |
| **Couchbase Lite** | モバイル特化だが、フルテキスト検索が主。ベクトル対応は重厚。 | **ONNX + int8 HNSWの密結合**による、推論から検索まで一気通貫の軽量パイプライン。 |
| **Chroma / Qdrant** | サーバーサイド（クラウド）前提。モバイル動作不可。 | 依存関係が極小。NDK（cdylib）ビルドでAndroidにネイティブ最適化済み。 |

**技術の堀（Moat）**:
Rustを用いた「Zero-Allocation JNIバインディング」、ONNXモデルの「int8量子化とHNSWグラフ内包メモリレイアウト」、および限られたサーマルバジェット内での「CPU/NPU適応的ルーティング」は、モバイル開発と低レイテンシシステムプログラミングの深い知見を要求するため、容易に模倣できない。

## 4. 評価される技術的アセット（Acqui-hireの対象）

Googleが評価・買収するのは単なるソースコードではなく、以下の問題を解決したチームの知見である。

1. **JNI / NDKにおけるZero-Copyアーキテクチャの実装能力**
2. **Androidの電力管理（Doze / WorkManager）とNNAPI推論の統合知見**
3. **Rustを用いたセキュアかつメモリ安全なシステムレベルコンポーネントの開発力**（Android OS自体がRustへの移行を進めている方針と完全一致）

## 5. マーケティング・露出戦略 (Maven Central & Dev Summit)

GoogleのDeveloper Relations / Android Coreチームの目に留まるためのマイルストーン。

1. **OSS公開とMaven Central配布**:
   - `io.github.nanorag:core:1.0.0` としてMaven Centralへデプロイ。
   - 「たった3行でAndroidにRAGを組み込める」をフックに、GitHubでスターを獲得。
2. **パフォーマンス・ホワイトペーパーの公開**:
   - Battery Historianを用いた消費電力エビデンス、Dalvikヒープ消費量ゼロの証明をZenn/Mediumで公開。
3. **Android Dev Summit / Google I/O へのCFP提出**:
   - 「Building On-Device RAG for Gemini Nano with Rust and ONNX」というテーマで登壇を狙う。
   - GoogleのDevRelエンジニア（Android MLチーム）に直接Twitter(X)/LinkedInでコンタクトし、テクニカルレビューを依頼する形でパイプを作る。
