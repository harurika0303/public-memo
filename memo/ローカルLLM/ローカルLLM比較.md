# ローカル LLM ツール比較

ローカル環境で LLM を動かす主要ツールの比較まとめです。

---

## ツール概要比較

| 項目               | Ollama                  | LM Studio               | llama.cpp                         | Jan                     |
| ------------------ | ----------------------- | ----------------------- | --------------------------------- | ----------------------- |
| 対象ユーザー       | 開発者・エンジニア      | 一般ユーザー〜開発者    | 上級者・開発者                    | 一般ユーザー            |
| UI                 | CLI 中心                | GUI 中心                | CLI のみ                          | GUI 中心                |
| セットアップ難易度 | 低                      | 低                      | 中〜高                            | 最低                    |
| ライセンス         | MIT                     | 商用利用: 要確認        | MIT                               | Apache 2.0              |
| 対応 OS            | Windows / macOS / Linux | Windows / macOS / Linux | Windows / macOS / Linux / Android | Windows / macOS / Linux |
| バックエンド       | llama.cpp               | llama.cpp / MLX         | 自身が本体                        | llama.cpp / MLX         |
| REST API           | あり（ポート 11434）    | あり（ポート 1234）     | あり（ポート 8080）               | あり（ポート 1337）     |
| OpenAI 互換 API    | あり                    | あり                    | あり                              | あり                    |
| Docker 対応        | あり                    | なし                    | あり                              | なし                    |
| MCP 対応           | なし（外部連携あり）    | あり                    | なし                              | あり                    |
| Web 検索           | なし                    | なし                    | なし                              | あり（Exa）             |
| エージェント機能   | なし（外部ツール連携）  | なし                    | なし                              | あり                    |
| Python SDK         | あり（公式）            | あり（公式）            | なし（非公式あり）                | なし（REST API 経由）   |
| GitHub Stars       | 100K+                   | —                       | 80K+                              | 40K+                    |

---

## 用途別おすすめ

### プログラミングが苦手 / とりあえず試したい

**Jan** がおすすめ。  
インストール後すぐに起動・チャットが可能。セットアップ作業なし。

### CLI が得意 / 開発環境に統合したい

**Ollama** がおすすめ。  
コマンド 1 行でモデルを起動でき、Docker との組み合わせも容易。  
REST API が安定しており、アプリへの組み込みがしやすい。

### GUI でモデルを管理したい / パラメータを細かく調整したい

**LM Studio** がおすすめ。  
Hugging Face 検索・ダウンロード・モデルパラメータ調整がすべて GUI で完結。

### 最大限のパフォーマンスを追求したい / 独自ビルドが必要

**llama.cpp** がおすすめ。  
量子化・GPU バックエンドの細かい制御、RISC-V / Vulkan 等の特殊環境にも対応。

---

## モデルファイル形式

| 形式        | 説明                               | 対応ツール                           |
| ----------- | ---------------------------------- | ------------------------------------ |
| GGUF        | llama.cpp 系の標準形式。量子化対応 | llama.cpp / Ollama / LM Studio / Jan |
| MLX         | Apple Silicon 向けの最適化形式     | LM Studio / Jan（macOS のみ）        |
| SafeTensors | HuggingFace 標準形式               | llama.cpp（変換必要）                |

---

## API エンドポイント比較

| ツール       | デフォルトポート | チャットエンドポイント                      |
| ------------ | ---------------- | ------------------------------------------- |
| Ollama       | 11434            | `http://localhost:11434/api/chat`           |
| LM Studio    | 1234             | `http://localhost:1234/v1/chat/completions` |
| llama-server | 8080             | `http://localhost:8080/v1/chat/completions` |
| Jan          | 1337             | `http://localhost:1337/v1/chat/completions` |

LM Studio / llama-server / Jan は OpenAI 互換なので、OpenAI の Python SDK からそのまま利用可能です。

```python
from openai import OpenAI

# 例: LM Studio に接続
client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

# 例: llama-server に接続
client = OpenAI(base_url="http://localhost:8080/v1", api_key="no-key")
```

---

## 動作要件の目安（モデルサイズ別）

| モデルサイズ  | 目安 RAM / VRAM | 代表的なモデル例                       |
| ------------- | --------------- | -------------------------------------- |
| 1B〜3B (Q4)   | 2〜4 GB         | Llama 3.2 1B/3B, Gemma 3 1B, TinyLlama |
| 7B〜8B (Q4)   | 6〜8 GB         | Llama 3.1 8B, Mistral 7B, Gemma 3 4B   |
| 13B〜14B (Q4) | 10〜14 GB       | Phi-4 14B, Mistral-Nemo 12B            |
| 27B〜32B (Q4) | 20〜24 GB       | Gemma 3 27B, Qwen3 32B                 |
| 70B (Q4)      | 40〜48 GB       | Llama 3.1 70B, Llama 3.3 70B           |

- GPU（NVIDIA / AMD / Apple Silicon）があれば VRAM が使われ高速化
- GPU がない場合は CPU + RAM で動作（速度は低下）
- Q4 = 4bit 量子化の場合の目安

---

## 各ツールのドキュメント

| ツール    | ドキュメント                                                  |
| --------- | ------------------------------------------------------------- |
| Ollama    | [memo/ローカルLLM/Ollamaについて.md](Ollamaについて.md)       |
| LM Studio | [memo/ローカルLLM/LM_Studioについて.md](LM_Studioについて.md) |
| llama.cpp | [memo/ローカルLLM/llama.cppについて.md](llama.cppについて.md) |
| Jan       | [memo/ローカルLLM/Janについて.md](Janについて.md)             |
