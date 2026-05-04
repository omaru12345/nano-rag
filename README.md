# nano-rag

スマホ内の文書（メモ・メッセージ・PDF）をローカルでベクトル化し、
オンデバイス LLM（Gemini Nano など）に文脈を供給するための **超軽量・低消費電力 RAG SDK**。

---

## 🎯 Goal: Googleに買収されること

このプロジェクトは「Google による Jetpac 型 acqui-hire（3〜5人 / 技術コア）」をエグジット目標とする。

**買収対象は SDK そのもの。** Android / Pixel チームが自社 OS にバンドルするレベルまで完成度を上げる。

| 想定買収先 | 統合シナリオ |
|---|---|
| **Android AI Core** | Gemini Nano に文脈を渡す標準 RAG レイヤー（OS API として開放）|
| **Pixel チーム** | Pixel の "魔法のような" 文脈アシスタント機能の基盤 |
| **Workspace モバイル** | Drive / Gmail / Docs のオフライン検索 |

### 評価される技術の堀（moat）
1. **消費電力**（連続インデックス時に 1 時間あたり 5%以下のバッテリー消費）
2. **インデックスサイズ**（10,000 文書で 50MB 以下）
3. **クエリレイテンシ**（top-10 検索を 50ms 以下）
4. **メモリ効率**（実行時 RAM ピーク 100MB 以下）

---

## 技術スタック

| レイヤー | 技術 |
|---|---|
| コア | Rust（cdylib として Android NDK / iOS にビルド） |
| 埋め込み | Sentence-Transformers の小型モデルを ONNX → 量子化（< 30MB） |
| ベクトル検索 | HNSW を Rust で再実装（or `instant-distance` を fork） |
| Android ラッパー | Kotlin + JNI |
| iOS ラッパー（後回し） | Swift + C インターフェイス |

---

## ディレクトリ構成

```
nano-rag/
├── core/                # Rust 実装（埋め込み + HNSW + ストレージ）
├── android/             # AAR ライブラリ（Kotlin + JNI）
├── bench/               # 消費電力 / レイテンシベンチ
└── docs/                # API ドキュメント / 買収ピッチ
```

---

## 3 フェーズ計画

### Phase 1（〜2か月）: コア実装
- [ ] Rust で埋め込み計算（ONNX Runtime 経由）
- [ ] HNSW でベクトル検索が動く
- [ ] 1,000 文書で end-to-end が動くデモ CLI

### Phase 2（〜4か月）: Android SDK
- [ ] AAR をローカルでビルド可能に
- [ ] サンプル Android アプリ（メモアプリ + RAG 検索）
- [ ] ベンチで消費電力・レイテンシを公表

### Phase 3（〜6か月）: 露出
- [ ] GitHub に OSS 公開（MIT / Apache 2.0）
- [ ] Maven Central に SDK 公開
- [ ] Android Dev Summit / Google I/O 周辺での露出

---

## 開発コマンド

```bash
# Rust コア
cd core && cargo build --release && cargo test

# Android SDK ビルド
cd android && ./gradlew assembleRelease

# ベンチ
cd bench && cargo run --release -- --dataset wiki1k
```

（実装は Phase 1 着手時に追加）
