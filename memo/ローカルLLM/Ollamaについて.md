# Ollama について

> 参照: [公式サイト](https://ollama.com/) / [GitHub README](https://github.com/ollama/ollama) / [公式ドキュメント](https://docs.ollama.com/)

---

## Ollama とは

Ollama は、ローカル環境でオープンソースの大規模言語モデル（LLM）を簡単に実行できるツールです。  
macOS・Windows・Linux に対応しており、コマンド 1 つでモデルのダウンロード〜実行が可能です。

**主な特徴:**

- データがローカルに留まり、外部に送信されない（プライバシー保護）
- REST API を通じてアプリケーションに組み込める
- 100 以上のオープンモデルに対応
- Python / JavaScript の公式 SDK あり
- Docker イメージも提供

---

## インストール

### Windows

PowerShell で実行:

```powershell
irm https://ollama.com/install.ps1 | iex
```

または [OllamaSetup.exe](https://ollama.com/download/OllamaSetup.exe) を手動ダウンロード。

### macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

または [Ollama.dmg](https://ollama.com/download/Ollama.dmg) を手動ダウンロード。

### Linux

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

手動インストール手順: [docs.ollama.com/linux](https://docs.ollama.com/linux#manual-install)

### Docker

```bash
docker run -d -p 11434:11434 ollama/ollama
```

Docker Hub: [hub.docker.com/r/ollama/ollama](https://hub.docker.com/r/ollama/ollama)

### パッケージマネージャー

| ツール              | コマンド                       |
| ------------------- | ------------------------------ |
| Homebrew (macOS)    | `brew install ollama`          |
| Pacman (Arch Linux) | `pacman -S ollama`             |
| Nix                 | nixpkgs の `ollama` パッケージ |

---

## クイックスタート

インストール後、ターミナルで `ollama` と入力するとインタラクティブメニューが表示されます。

```bash
ollama
```

`↑/↓` でナビゲート、`Enter` で起動、`→` でモデル変更、`Esc` で終了。

**すぐにモデルと会話する:**

```bash
ollama run gemma3
```

---

## CLI コマンド一覧

| コマンド                     | 説明                               |
| ---------------------------- | ---------------------------------- |
| `ollama run <モデル名>`      | モデルを起動してチャット           |
| `ollama pull <モデル名>`     | モデルをダウンロード               |
| `ollama rm <モデル名>`       | モデルを削除                       |
| `ollama ls`                  | ダウンロード済みモデルを一覧表示   |
| `ollama ps`                  | 実行中のモデルを一覧表示           |
| `ollama stop <モデル名>`     | 実行中のモデルを停止               |
| `ollama serve`               | Ollama サーバーを起動              |
| `ollama create -f Modelfile` | カスタムモデルを作成               |
| `ollama launch <ツール名>`   | 外部ツール（Claude Code 等）を起動 |

### モデルの実行例

```bash
# 通常のチャット
ollama run gemma3

# 画像を含む質問（マルチモーダルモデル）
ollama run gemma3 "この画像は何ですか？ /path/to/image.png"

# 複数行入力
>>> """こんにちは、
... 世界！
... """
```

### カスタムモデルの作成

`Modelfile` を作成し、`ollama create` で登録できます。

```
FROM gemma3
SYSTEM """あなたは親切な日本語アシスタントです。"""
```

```bash
ollama create my-assistant -f Modelfile
ollama run my-assistant
```

---

## 主要モデル一覧

[ollama.com/library](https://ollama.com/library) で全モデルを確認できます。

### 汎用モデル

| モデル        | パラメータ      | 特徴                                        |
| ------------- | --------------- | ------------------------------------------- |
| `gemma3`      | 1B〜27B         | Google製。シングルGPUで動作する最高クラス   |
| `llama3.1`    | 8B / 70B / 405B | Meta製。高性能な汎用モデル                  |
| `llama3.2`    | 1B / 3B         | Meta製。軽量版                              |
| `qwen3`       | 0.6B〜235B      | Alibaba製。ツール呼び出し対応・推論モード有 |
| `mistral`     | 7B              | Mistral AI製。バランスの良い汎用モデル      |
| `phi4`        | 14B             | Microsoft製。高性能な中規模モデル           |
| `deepseek-r1` | 1.5B〜671B      | 推論特化モデル。o3相当の性能                |

### コード生成モデル

| モデル           | パラメータ | 特徴                            |
| ---------------- | ---------- | ------------------------------- |
| `qwen2.5-coder`  | 0.5B〜32B  | コーディング特化。199タグと人気 |
| `codellama`      | 7B〜70B    | Meta製コード生成モデル          |
| `deepseek-coder` | 1.3B〜33B  | コード特化・高性能              |

### 埋め込みモデル（RAG用）

| モデル              | パラメータ | 特徴                   |
| ------------------- | ---------- | ---------------------- |
| `nomic-embed-text`  | -          | 高性能テキスト埋め込み |
| `mxbai-embed-large` | 335M       | mixedbread.ai製        |
| `bge-m3`            | 567M       | 多機能・多言語対応     |

### マルチモーダルモデル（画像対応）

| モデル            | パラメータ | 特徴                  |
| ----------------- | ---------- | --------------------- |
| `llava`           | 7B〜34B    | 画像＋テキスト理解    |
| `llama3.2-vision` | 11B / 90B  | Llama 3.2の画像対応版 |
| `gemma3`          | 4B〜27B    | ビジョン対応          |

---

## REST API

インストール後、Ollama は `http://localhost:11434` で API を提供します。

### ベース URL

```
http://localhost:11434/api
```

### テキスト生成

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3",
  "prompt": "空はなぜ青いのですか？"
}'
```

### チャット

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "gemma3",
  "messages": [
    { "role": "user", "content": "こんにちは！" }
  ],
  "stream": false
}'
```

詳細は [docs.ollama.com/api](https://docs.ollama.com/api) を参照。

---

## SDK の利用

### Python

```bash
pip install ollama
```

```python
from ollama import chat

response = chat(
    model='gemma3',
    messages=[
        {'role': 'user', 'content': '空はなぜ青いのですか？'}
    ]
)
print(response.message.content)
```

### JavaScript / Node.js

```bash
npm i ollama
```

```javascript
import ollama from 'ollama';

const response = await ollama.chat({
  model: 'gemma3',
  messages: [{ role: 'user', content: '空はなぜ青いのですか？' }],
});
console.log(response.message.content);
```

---

## 外部ツールとの連携

`ollama launch` コマンドで各種ツールと連携できます。

```bash
# Claude Code と連携
ollama launch claude

# Codex と連携
ollama launch codex

# OpenCode と連携
ollama launch opencode

# 特定モデルを指定して起動
ollama launch claude --model qwen3
```

### コードエディタとの統合

| ツール                                              | 説明                                                     |
| --------------------------------------------------- | -------------------------------------------------------- |
| [Continue](https://github.com/continuedev/continue) | VS Code / JetBrains 対応のオープンソース AI コード補助   |
| [Cline](https://github.com/cline/cline)             | VS Code 拡張。複数ファイル・リポジトリ単位のコーディング |
| AI Toolkit for VS Code                              | Microsoft 公式 VS Code 拡張                              |
| [twinny](https://github.com/rjmacarthy/twinny)      | GitHub Copilot の代替 VS Code 拡張                       |

### チャット UI

| ツール                                                       | 説明                                     |
| ------------------------------------------------------------ | ---------------------------------------- |
| [Open WebUI](https://github.com/open-webui/open-webui)       | セルフホスト型 AI チャット UI            |
| [AnythingLLM](https://github.com/Mintplex-Labs/anything-llm) | デスクトップ向けオールインワン AI アプリ |
| [LibreChat](https://github.com/danny-avila/LibreChat)        | マルチプロバイダー対応 ChatGPT クローン  |

---

## 動作要件の目安

| モデルサイズ        | 推奨 RAM/VRAM |
| ------------------- | ------------- |
| 1B〜3B パラメータ   | 4GB 以上      |
| 7B〜8B パラメータ   | 8GB 以上      |
| 13B〜14B パラメータ | 16GB 以上     |
| 30B〜32B パラメータ | 32GB 以上     |
| 70B パラメータ      | 64GB 以上     |

- GPU（NVIDIA / AMD）がある場合は GPU メモリ（VRAM）が優先使用される
- GPU がない場合は CPU+RAM で動作（低速）

---

## コミュニティ・サポート

- [Discord](https://discord.gg/ollama)
- [Reddit r/ollama](https://reddit.com/r/ollama)
- [GitHub Issues](https://github.com/ollama/ollama/issues)
- [X (Twitter) @ollama](https://x.com/ollama)
