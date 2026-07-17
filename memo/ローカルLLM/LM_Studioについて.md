# LM Studio について

> 参照: [公式サイト](https://lmstudio.ai/) / [公式ドキュメント](https://lmstudio.ai/docs)

---

## LM Studio とは

LM Studio は、GUI（グラフィカルインターフェース）ベースのローカル LLM 実行ツールです。  
Hugging Face からモデルを検索・ダウンロードし、チャット画面ですぐ使えます。  
OpenAI 互換の REST API サーバーも内蔵しており、既存アプリへの組み込みも容易です。

**主な特徴:**

- GUI でモデルの検索・ダウンロード・管理ができる（プログラミング不要）
- OpenAI 互換 API を提供（既存ツールをそのまま接続可能）
- RAG（ドキュメントとのチャット）をオフラインで実現
- MCP サーバーとの連携対応
- GUI なしのヘッドレスモード（`llmster`）も利用可能
- Python / TypeScript SDK あり

---

## システム要件

### macOS

| 項目   | 要件                                            |
| ------ | ----------------------------------------------- |
| チップ | Apple Silicon (M1/M2/M3/M4)                     |
| OS     | macOS 14.0 以上                                 |
| RAM    | 16GB 以上推奨（8GB でも動作するが小モデル限定） |
| ※      | Intel Mac は現在非対応                          |

### Windows

| 項目           | 要件                              |
| -------------- | --------------------------------- |
| アーキテクチャ | x64 / ARM (Snapdragon X Elite)    |
| CPU            | AVX2 命令セット対応（x64 の場合） |
| RAM            | 16GB 以上推奨                     |
| GPU            | VRAM 4GB 以上推奨                 |

### Linux

| 項目           | 要件              |
| -------------- | ----------------- |
| アーキテクチャ | x64 / ARM64       |
| 配布形式       | AppImage          |
| OS             | Ubuntu 20.04 以上 |

---

## インストール

[lmstudio.ai/download](https://lmstudio.ai/download) からインストーラーをダウンロードして実行するだけです。

- **macOS**: `.dmg` ファイル
- **Windows**: `.exe` インストーラー
- **Linux**: `.AppImage` ファイル

---

## 基本的な使い方（GUI）

### モデルのダウンロード

1. LM Studio を起動
2. 左サイドバーの「Search」タブを開く
3. モデル名（例: `llama3.2`）を検索
4. ダウンロードしたいモデルを選択して「Download」

### チャットの開始

1. 「Chat」タブを開く
2. 上部のモデル選択からダウンロード済みモデルを選択
3. メッセージを入力して送信

### ドキュメントとのチャット（RAG）

1. チャット画面でファイルアイコンをクリック
2. PDF や テキストファイルを添付
3. 「その文書について教えて」などと質問する

---

## CLI ツール（`lms`）

`lms` は LM Studio に同梱の CLI ツールです。LM Studio を一度起動すれば自動的に使えるようになります。

```bash
lms --help
```

### 主要コマンド一覧

| コマンド              | 説明                             |
| --------------------- | -------------------------------- |
| `lms chat`            | ターミナルでモデルとチャット     |
| `lms get <モデル名>`  | モデルを検索・ダウンロード       |
| `lms ls`              | ダウンロード済みモデルを一覧表示 |
| `lms ps`              | メモリにロード中のモデルを表示   |
| `lms load <モデル名>` | モデルをロード                   |
| `lms unload [--all]`  | モデルをアンロード               |
| `lms server start`    | ローカルサーバーを起動           |
| `lms server stop`     | ローカルサーバーを停止           |
| `lms server status`   | サーバーの状態を確認             |

### モデルのロード例

```bash
# GPU 使用率を指定してロード（1.0 = 100%）
lms load llama3.2 --gpu=max

# カスタム識別子を付けてロード
lms load openai/gpt-oss-20b --identifier="my-model-name"

# コンテキスト長を指定
lms load llama3.2 --context-length=8192
```

---

## REST API（OpenAI 互換）

サーバー起動後、`http://localhost:1234` で OpenAI 互換 API を利用できます。

### サーバーの起動

```bash
lms server start
```

または GUI の「Local Server」タブから起動。

### チャット補完エンドポイント

```bash
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama3.2",
    "messages": [
      {"role": "user", "content": "こんにちは！"}
    ]
  }'
```

### Python SDK

```bash
pip install lmstudio
```

```python
import lmstudio

client = lmstudio.Client()
response = client.chat.completions.create(
    model="llama3.2",
    messages=[{"role": "user", "content": "こんにちは！"}]
)
print(response.choices[0].message.content)
```

### TypeScript / Node.js SDK

```bash
npm install lmstudio
```

```typescript
import { LMStudioClient } from 'lmstudio';

const client = new LMStudioClient();
const response = await client.llm.respond([
  { role: 'user', content: 'こんにちは！' }
]);
console.log(response.content);
```

---

## MCP サーバーとの連携

LM Studio は MCP（Model Context Protocol）クライアントとして動作します。

1. GUI の「Plugins」タブを開く
2. 「Add MCP Server」から接続設定を追加
3. MCP ツールをモデルと一緒に使用

---

## 外部ツールとの連携

| ツール      | 説明                                    |
| ----------- | --------------------------------------- |
| Claude Code | Anthropic のコーディングエージェント    |
| Codex       | OpenAI のコーディングアシスタント       |
| OpenClaw    | Ollama/LM Studio 対応の AI アシスタント |

---

## ヘッドレスモード（GUI なし）

`llmster` を使うとデスクトップアプリなしでサーバーとして動作します。  
サーバー・CI 環境での利用に適しています。

詳細: [docs.ollama.com/developer/core/headless](https://lmstudio.ai/docs/developer/core/headless)

---

## コミュニティ・サポート

- [Discord](https://discord.gg/aPQfnNkxGC)
- [GitHub Issues](https://github.com/lmstudio-ai/lmstudio-bug-tracker)
