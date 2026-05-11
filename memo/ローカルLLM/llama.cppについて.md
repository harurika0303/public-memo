# llama.cpp について

> 参照: [GitHub リポジトリ](https://github.com/ggml-org/llama.cpp)

---

## llama.cpp とは

llama.cpp は、C/C++ で実装された高速 LLM 推論エンジンです。  
Georgi Gerganov 氏が開発し、依存関係ゼロの軽量設計が特徴です。  
Ollama や LM Studio などの多くのツールが内部バックエンドとして利用しています。

**主な特徴:**

- 純粋な C/C++ 実装で依存関係なし
- 幅広いハードウェアに対応（CPU・GPU・Apple Silicon・RISC-V 等）
- 量子化（1.5bit〜8bit）による省メモリ・高速推論
- NVIDIA / AMD / Apple Silicon / Vulkan など多様な GPU バックエンド
- OpenAI 互換の HTTP サーバー（`llama-server`）を内蔵
- GGUF 形式のモデルファイルを使用

---

## インストール

### 方法 1: パッケージマネージャー（推奨・簡単）

```bash
# macOS (Homebrew)
brew install llama.cpp

# Windows (winget)
winget install llama.cpp

# Nix
nix-env -iA nixpkgs.llama-cpp
```

### 方法 2: プレビルドバイナリ

[GitHub Releases](https://github.com/ggml-org/llama.cpp/releases) から OS に合ったバイナリをダウンロードして展開するだけです。

### 方法 3: Docker

```bash
# CPU のみ
docker run -it ghcr.io/ggml-org/llama.cpp:latest

# CUDA (NVIDIA GPU)
docker run --gpus all -it ghcr.io/ggml-org/llama.cpp:latest-cuda
```

### 方法 4: ソースからビルド

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release
```

GPU サポートを有効にする場合（CUDA）:

```bash
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```

---

## 対応バックエンド

| バックエンド | 対応ハードウェア      |
| ------------ | --------------------- |
| Metal        | Apple Silicon         |
| CUDA         | NVIDIA GPU            |
| HIP          | AMD GPU               |
| Vulkan       | 汎用 GPU              |
| SYCL         | Intel / NVIDIA GPU    |
| OpenVINO     | Intel CPU / GPU / NPU |
| CANN         | Ascend NPU            |
| OpenCL       | Adreno GPU            |
| BLAS / BLIS  | CPU 全般              |
| RPC          | 分散推論              |

---

## モデルの取得

### Hugging Face からダウンロード（最も簡単）

`-hf` フラグで Hugging Face から直接ダウンロード＆実行:

```bash
# Gemma 3 1B をダウンロードして実行
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF
```

### 手動ダウンロード

[Hugging Face](https://huggingface.co/models?library=gguf&sort=trending) で GGUF ファイルを検索してダウンロード。  
トレンドモデルや LLaMA 系は以下で確認:

- [トレンドモデル](https://huggingface.co/models?library=gguf&sort=trending)
- [LLaMA GGUF](https://huggingface.co/models?sort=trending&search=llama+gguf)

### 他形式からの変換

SafeTensors / PyTorch 形式のモデルを GGUF に変換:

```bash
# Python スクリプトで変換
python convert_hf_to_gguf.py ./my-model-dir --outtype f16
```

Hugging Face のオンラインツールも利用可能:
- [GGUF-my-repo](https://huggingface.co/spaces/ggml-org/gguf-my-repo) — ブラウザ上で変換・量子化

---

## モデルの量子化

量子化によりモデルサイズを圧縮し、推論を高速化できます。

| 量子化レベル | 精度 | ファイルサイズ | 用途               |
| ------------ | ---- | -------------- | ------------------ |
| Q8_0         | 高   | 大             | 精度優先           |
| Q6_K         | 高め | 中大           | バランス良好       |
| Q5_K_M       | 中高 | 中             | 推奨バランス       |
| Q4_K_M       | 中   | 小中           | 最も一般的         |
| Q3_K_M       | 低め | 小             | VRAM 節約          |
| Q2_K         | 低   | 最小           | 超軽量デバイス向け |

```bash
# Q4_K_M に量子化
./build/bin/llama-quantize ./model.gguf ./model-q4km.gguf Q4_K_M
```

---

## CLI ツール

### `llama-cli` — インタラクティブチャット

```bash
# ローカルの GGUF ファイルを指定
llama-cli -m ./model.gguf

# Hugging Face から直接起動
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF

# 会話モードを明示的に有効化
llama-cli -m model.gguf -cnv

# コンテキスト長を指定
llama-cli -m model.gguf -c 4096

# GPU レイヤー数を指定（-1 で全層を GPU に）
llama-cli -m model.gguf -ngl -1
```

### `llama-server` — OpenAI 互換 HTTP サーバー

```bash
# デフォルト設定でサーバー起動（ポート 8080）
llama-server -m model.gguf --port 8080

# Hugging Face から直接起動
llama-server -hf ggml-org/gemma-3-1b-it-GGUF

# Web UI: http://localhost:8080
# API:    http://localhost:8080/v1/chat/completions
```

API の使用例:

```bash
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3",
    "messages": [
      {"role": "user", "content": "こんにちは！"}
    ]
  }'
```

### その他のツール

| ツール             | 説明                               |
| ------------------ | ---------------------------------- |
| `llama-bench`      | モデルのパフォーマンスベンチマーク |
| `llama-perplexity` | モデルのテキスト品質測定           |
| `llama-quantize`   | モデルの量子化                     |
| `llama-simple`     | 開発者向けの最小実装サンプル       |

---

## 主要パラメータ

| パラメータ               | 説明                                        |
| ------------------------ | ------------------------------------------- |
| `-m <path>`              | モデルファイルのパス                        |
| `-hf <user/model>`       | Hugging Face からモデルを指定               |
| `-ngl <n>`               | GPU オフロードするレイヤー数（`-1` で全層） |
| `-c <n>`                 | コンテキスト長（デフォルト: 512）           |
| `-n <n>`                 | 生成するトークン数                          |
| `-t <n>`                 | CPU スレッド数                              |
| `--temp <f>`             | 温度パラメータ（創造性の調整）              |
| `-cnv`                   | 会話モードを有効化                          |
| `--chat-template <name>` | チャットテンプレートの指定                  |
| `-p <prompt>`            | プロンプトを直接指定                        |

---

## マルチ GPU での利用

複数の GPU を使う場合は `--tensor-split` オプションで負荷分散できます。

```bash
# 2つの GPU に均等分割
llama-server -m model.gguf --tensor-split 0.5,0.5 -ngl 999
```

詳細: [Multi-GPU usage](https://github.com/ggml-org/llama.cpp/blob/master/docs/multi-gpu.md)

---

## VS Code 連携

llama.cpp チームが提供する VS Code 拡張（FIM 補完）:  
[github.com/ggml-org/llama.vscode](https://github.com/ggml-org/llama.vscode)

---

## 動作要件の目安

| モデルサイズ | 推奨 RAM/VRAM |
| ------------ | ------------- |
| 1B〜3B (Q4)  | 2GB 以上      |
| 7B〜8B (Q4)  | 6GB 以上      |
| 13B (Q4)     | 10GB 以上     |
| 30B (Q4)     | 20GB 以上     |
| 70B (Q4)     | 40GB 以上     |

---

## コミュニティ・サポート

- [GitHub Issues](https://github.com/ggml-org/llama.cpp/issues)
- [GitHub Discussions](https://github.com/ggml-org/llama.cpp/discussions)
- [Reddit r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/)
