# Jan について

> 参照: [公式サイト](https://jan.ai/) / [公式ドキュメント](https://www.jan.ai/docs)

---

## Jan とは

Jan は、Claude や ChatGPT のオープンソース代替として設計されたローカル LLM アプリです。  
シンプルで美しい UI と、ローカル完結型のプライバシー重視設計が特徴で、550 万回以上ダウンロードされています（2026年5月時点）。

**主な特徴:**

- セットアップ不要（初回起動時にデフォルトモデルを自動ダウンロード）
- ローカル・クラウドモデルの両方に対応
- オフライン動作（データが一切外部に送信されない）
- Web 検索機能を内蔵（Exa による無料提供）
- MCP（Model Context Protocol）サーバー対応
- 自律エージェント機能（ファイル読み取り・カレンダー管理・Slack/Discord 操作など）
- OpenAI 互換のローカル API サーバーを内蔵
- Apache 2.0 ライセンス（完全オープンソース）

---

## インストール

[jan.ai/download](https://jan.ai/download) からインストーラーをダウンロードして実行します。

| OS      | ダウンロード先        |
| ------- | --------------------- |
| Windows | `.exe` インストーラー |
| macOS   | `.dmg` ファイル       |
| Linux   | `.AppImage` / `.deb`  |

---

## クイックスタート

### 1. インストール後に起動

Jan を起動すると、デフォルトモデルが自動的にダウンロードされます。  
設定は不要です。

### 2. チャット開始

モデルのダウンロードが完了したらすぐにチャットができます。

試しに聞いてみる例:
- 「量子コンピュータをわかりやすく説明して」
- 「Pythonでリストをソートする関数を書いて」

### 3. Web 検索

Jan はデフォルトで Web 検索が有効になっています（Exa によるリアルタイム検索）。

---

## Jan 独自モデル（Jan Models）

Jan チームが開発・最適化した専用モデルが利用できます。

| モデル          | 特徴                                    |
| --------------- | --------------------------------------- |
| `Jan-Code-4B`   | コード生成・推論に最適化                |
| `Jan-v3-4B`     | 汎用インストラクションモデル            |
| `Jan-v2-VL`     | 画像理解対応のビジョン言語モデル        |
| `Jan-v1`        | 初代 Jan 独自モデル                     |
| `Jan Nano 128k` | 超長文コンテキスト対応（128k トークン） |
| `Jan Nano 32k`  | 軽量・32k コンテキスト対応              |

---

## 対応モデル（サードパーティ）

Jan は外部モデルとも連携できます。

### ローカルモデル（GGUF / MLX 形式）

Hugging Face からダウンロードした GGUF モデルを「Hub」タブからインポート可能。

| モデル系統   | 代表的なモデル                    |
| ------------ | --------------------------------- |
| Meta Llama   | `llama3.2`, `llama3.1` など       |
| Google Gemma | `gemma3`, `gemma2` など           |
| Alibaba Qwen | `qwen3`, `qwen2.5` など           |
| Mistral AI   | `mistral`, `mistral-small` など   |
| Microsoft    | `phi4`, `phi3` など               |
| DeepSeek     | `deepseek-r1`, `deepseek-v3` など |

### クラウドモデル（API キー接続）

設定 → プロバイダーから API キーを設定することでクラウドモデルも利用できます。

| プロバイダー | モデル例                     |
| ------------ | ---------------------------- |
| OpenAI       | GPT-4o, o3 など              |
| Anthropic    | Claude 3.5 Sonnet など       |
| Google       | Gemini 1.5 Pro など          |
| Mistral AI   | Mistral Large など           |
| Groq         | 高速推論 API                 |
| OpenRouter   | 多数のモデルへのゲートウェイ |
| Hugging Face | Hugging Face Pro モデル      |

---

## 主な機能

### Projects（プロジェクト）

会話を目的別に整理できます。プロジェクトごとに共通の指示やファイルを設定可能。

### Assistants（アシスタント）

用途に応じてパーソナライズされた AI アシスタントを作成できます。

### Agents（エージェント）

自律的に動作するエージェントを設定できます。できることの例:
- ファイルの読み取り・書き込み
- カレンダーの管理
- WhatsApp / Discord / Slack でのメッセージ送受信

### MCP サーバー連携

MCP（Model Context Protocol）対応のサーバーを接続することで、モデルの機能を拡張できます:
- Web 検索
- コード実行
- データベースアクセス

---

## ローカル API サーバー

Jan はバックグラウンドで OpenAI 互換 API サーバーを提供します。

### エンドポイント

```
http://localhost:1337/v1
```

### チャット補完

```bash
curl http://localhost:1337/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "jan-v1",
    "messages": [
      {"role": "user", "content": "こんにちは！"}
    ]
  }'
```

---

## CLI ツール

```bash
# Jan CLI でモデルを起動
jan chat

# 利用可能なモデルを表示
jan models list

# エージェントを起動
jan agent start
```

詳細: [jan.ai/docs/desktop/cli](https://www.jan.ai/docs/desktop/cli)

---

## 推論エンジン

| エンジン  | 説明                                   |
| --------- | -------------------------------------- |
| llama.cpp | Windows / Linux / macOS (Intel) で使用 |
| MLX       | macOS Apple Silicon 向けの高速エンジン |

設定 → 「Local AI Engine」タブからエンジンを切り替えられます。

---

## 外部ツールとの連携

| ツール      | 説明                                                      |
| ----------- | --------------------------------------------------------- |
| Claude Code | Anthropic のコーディングエージェントを Jan モデルで動かす |
| OpenClaw    | Ollama/Jan 対応の AI アシスタント                         |
| MCP Servers | 拡張ツールとの接続                                        |

---

## プライバシー方針

- ローカルモデル使用時はデータが一切外部に送信されない
- Jan のトレーニングデータとしてユーザーデータは使用されない
- [プライバシーポリシー](https://www.jan.ai/docs/desktop/privacy)

---

## コミュニティ・サポート

- [Discord](https://discord.com/invite/FTk2MvZwJH)
- [Reddit r/askjan](https://www.reddit.com/r/askjan/)
- [GitHub (41.9K stars)](https://github.com/janhq/jan)
- [Hugging Face (123 モデル)](https://huggingface.co/janhq)
