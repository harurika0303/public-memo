# Qwen 3.5 9B をローカルで使う手順

## モデル概要

| 項目 | 内容 |
|---|---|
| モデル名 | Qwen 3.5 9B |
| 開発元 | Alibaba（Qwen チーム） |
| パラメータ数 | 9B |
| ファイルサイズ | 約 6.6 GB（Q4_K_M 量子化） |
| コンテキスト長 | 256K トークン（Llama 3.1 8B の 2 倍） |
| 対応入力 | テキスト・画像（マルチモーダル） |
| Thinking モード | あり（推論ステップを出力する機能） |
| ツール呼び出し | あり |
| 知識カットオフ | 2025 年頃 |

> 出典: [Ollama - qwen3.5](https://ollama.com/library/qwen3.5)

### Llama 3.1 8B との比較

| 項目 | Llama 3.1 8B | Qwen 3.5 9B |
|---|---|---|
| サイズ | 4.7 GB | 6.6 GB |
| コンテキスト | 128K | 256K |
| 画像入力 | ❌ | ✅ |
| Thinking モード | ❌ | ✅ |
| 日本語性能 | 普通 | 高め（201 言語対応） |
| ツール呼び出し | ✅ | ✅ |

---

## Step 1: モデルをダウンロード

Ollama がインストール済みであることが前提です（[Ollama 導入手順](Ollamaについて.md) 参照）。

```bash
ollama pull qwen3.5:9b
```

ダウンロードサイズ: 約 6.6 GB。完了まで数分〜数十分かかります。

ダウンロード確認:

```bash
ollama list
```

`qwen3.5:9b` が表示されれば OK。

---

## Step 2: 動作確認

### チャット（CLI）

```bash
ollama run qwen3.5:9b
```

`>>> ` プロンプトが表示されたら成功。終了は `/bye`。

### API で確認（bash / Git Bash）

```bash
curl http://localhost:11434/api/chat -d '{"model": "qwen3.5:9b", "messages": [{"role": "user", "content": "Hello"}], "stream": false}'
```

### Thinking モードを試す

Thinking モード（推論過程を出力）を有効にする場合は、システムプロンプトまたはモデル名に `:thinking` タグを付けます:

```bash
ollama run qwen3.5:9b
>>> /set system "Think step by step before answering."
>>> 日本語で、1から10までの素数をすべて答えてください
```

API の場合:

```bash
curl http://localhost:11434/api/chat -d '{"model": "qwen3.5:9b", "messages": [{"role": "user", "content": "1から10の素数は？"}], "think": true, "stream": false}'
```

`think: true` を付けると `<think>...</think>` タグ内に推論過程が含まれます。

---

## Step 3: Continue（VS Code 拡張）に追加する

既存の `C:\Users\<ユーザー名>\.continue\config.yaml` に追記します。

### config.yaml を編集

`models:` セクションに Qwen 3.5 9B のエントリを追加:

```yaml
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: Llama 3.1 8B (Local)
    provider: ollama
    model: llama3.1:8b
    apiBase: http://localhost:11434

  - name: Qwen 3.5 9B (Local)
    provider: ollama
    model: qwen3.5:9b
    apiBase: http://localhost:11434

tabAutocompleteModel:
  name: Qwen 3.5 9B (Local)
  provider: ollama
  model: qwen3.5:9b
  apiBase: http://localhost:11434
```

> **ポイント**: `tabAutocompleteModel` を Qwen 3.5 9B に変更すると、補完品質が向上する場合があります。

### 設定反映

ファイルを保存（`Ctrl+S`）後、VS Code をリロード:

```
Ctrl+Shift+P → Developer: Reload Window
```

Continue のサイドバーでモデルを切り替えられるようになります。

---

## Thinking モードについて

Qwen 3.5 はモデル内部に「考える」ステップを持っています。難しい質問（数学、コード設計、論理問題）では Thinking モードを有効にすると回答の質が上がります。

通常モードと Thinking モードの使い分け:

| 用途 | モード |
|---|---|
| 簡単な質問・コード補完 | 通常（速い） |
| 複雑なロジック・設計相談 | Thinking（遅いが高精度） |
| コードのデバッグ | Thinking |
| 翻訳・要約 | 通常 |

---

## トラブルシューティング

| 症状 | 対処 |
|---|---|
| ダウンロードが止まる | `ollama pull qwen3.5:9b` を再実行（再開される） |
| メモリ不足でクラッシュ | `qwen3.5:4b`（3.4 GB）に変更する |
| Continue でモデルが出ない | `ollama list` でモデル名確認、config.yaml の `model:` と一致させる |
| 応答が英語になる | プロンプトの先頭に「日本語で答えてください。」を追加する |

---

## 参考リンク

- [Ollama - qwen3.5](https://ollama.com/library/qwen3.5)
- [Qwen 公式 GitHub](https://github.com/QwenLM/Qwen3.5)
- [Artificial Analysis - Qwen3.5 9B](https://artificialanalysis.ai/models/qwen3-5-9b)
