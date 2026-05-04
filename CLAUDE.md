# CLAUDE.md — nano-rag

## プロジェクト概要

オンデバイス RAG SDK。**Rust コア + Android (Kotlin) ラッパー**。
ゴール: Google（Android / Pixel）による acqui-hire。詳細は README 参照。

## 設計原則

- **メモリ・電力第一**: 機能追加よりも消費リソースの削減を優先
- **依存最小**: Rust 側のクレートは厳選、不要なものは入れない
- **データ流出ゼロ**: ネットワーク呼び出しはコアに含めない
- **API は安定**: SDK として公開する以上、破壊的変更は慎重に

## ディレクトリ構成

`core/` が Rust、`android/` が Kotlin/JNI ラッパー、`bench/` が計測。
詳細は README 参照。

## ブランチ命名

```
feature-{name}-{description}
bug-{name}-{description}
refactoring-{name}-{description}
```

メインブランチは `main`、PR は `main` に向ける。

## コミット前チェック

- Rust: `cargo test` + `cargo clippy -- -D warnings` + `cargo fmt --check`
- Android: `./gradlew test`
- ベンチが回る変更ならローカルベンチ実行

## やらないこと

- LLM 本体の実装（Gemini Nano に任せる）
- iOS 対応（Phase 4 以降。最初は Android に集中）
- クラウド同期機能（オンデバイスを徹底）
