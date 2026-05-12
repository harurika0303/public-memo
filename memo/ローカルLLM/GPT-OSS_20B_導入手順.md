# GPT-OSS 20B をローカルで使う手順

## モデル概要

| 項目                       | 内容                        |
| -------------------------- | --------------------------- |
| モデル名                   | gpt-oss-20b                 |
| 開発元                     | OpenAI                      |
| リリース日                 | 2025年8月5日                |
| 総パラメータ数             | 21B（MoE構造）              |
| 推論時アクティブパラメータ | **3.6B**（MoEのため軽い）   |
| ファイルサイズ             | **14 GB**（MXFP4量子化）    |
| コンテキスト長             | 128K トークン               |
| 推論モデル                 | ✅ あり（低・中・高の3段階） |
| ツール呼び出し             | ✅ あり                      |
| ライセンス                 | Apache 2.0（商用利用可）    |
| 知識カットオフ             | 2024年5月                   |

> 出典: [OpenAI 公式発表](https://openai.com/ja-JP/index/introducing-gpt-oss/) / [Artificial Analysis](https://artificialanalysis.ai/models/gpt-oss-20b) / [Ollama - gpt-oss](https://ollama.com/library/gpt-oss)

---

## ⚠️ 動作要件について（重要）

OpenAI 公式の最小要件は **16GB RAM（またはVRAM）** です。

| モデル                   | 必要メモリ     | 参考                     |
| ------------------------ | -------------- | ------------------------ |
| Llama 3.1 8B（Q4_K_M）   | 約 5〜6 GB     | このPCで動作確認済み     |
| Qwen 3.5 9B（Q4_K_M）    | 約 7〜8 GB     | 試験中                   |
| **gpt-oss-20b（MXFP4）** | **最小 16 GB** | ギリギリ〜動かない可能性 |

このPCは Llama 3.1 8B で "BARELY RUNS"（ギリギリ動作）の判定のため、gpt-oss-20b は **動作しない、または非常に遅くなる可能性が高い**です。

それでも試す場合は以下の手順で進めてください。

---

## Step 1: Ollama を最新版に更新する

gpt-oss は MXFP4 という新しい量子化形式を使用しており、**古い Ollama では動作しません**。

現在のバージョン確認:

```powershell
ollama --version
```

更新（PowerShell）:

```powershell
winget upgrade Ollama.Ollama
```

または [https://ollama.com/download](https://ollama.com/download) から最新版を手動インストール。

---

## Step 2: モデルをダウンロード

```bash
ollama pull gpt-oss:20b
```

ダウンロードサイズ: **約 14 GB**。完了まで数十分〜1時間程度かかる場合があります。

ダウンロード確認:

```bash
ollama list
```

`gpt-oss:20b` が表示されれば OK。

---

## Step 3: 動作確認

### CLI でチャット

```bash
ollama run gpt-oss:20b
```

> **このPCで起動しない場合** は Step 4 の代替手段を参照してください。

### API で確認（bash / Git Bash）

```bash
curl http://localhost:11434/api/chat -d '{"model": "gpt-oss:20b", "messages": [{"role": "user", "content": "Hello"}], "stream": false}'
```

> 初回はモデルのロードに **数十秒〜数分** かかる場合があります。

---

## Step 4: 推論レベルの調整（reasoning_effort）

gpt-oss の大きな特徴は **推論レベルを調整できる**ことです。低に設定すると速くなります。

システムプロンプトで設定:

```
reasoning_effort: low
```

または

```
reasoning_effort: medium
```

CLI での使用例:

```bash
ollama run gpt-oss:20b
>>> /set system "reasoning_effort: low"
>>> 日本語で、1から10までの素数を答えてください
```

| レベル   | 速度 | 品質 |
| -------- | ---- | ---- |
| `low`    | 速い | 普通 |
| `medium` | 中間 | 高い |
| `high`   | 遅い | 最高 |

---

## Step 5: Continue（VS Code 拡張）に追加する

既存の `C:\Users\<ユーザー名>\.continue\config.yaml` に追記します:

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

  - name: GPT-OSS 20B (Local)
    provider: ollama
    model: gpt-oss:20b
    apiBase: http://localhost:11434

tabAutocompleteModel:
  name: Llama 3.1 8B (Local)
  provider: ollama
  model: llama3.1:8b
  apiBase: http://localhost:11434
```

> **ポイント**: `tabAutocompleteModel` には軽い Llama 3.1 8B を残すことを推奨。gpt-oss-20b は重いため補完には不向きです。

---

## モデルの特徴と他モデルとの比較

| 項目                      | Llama 3.1 8B | Qwen 3.5 9B   | GPT-OSS 20B |
| ------------------------- | ------------ | ------------- | ----------- |
| ファイルサイズ            | 4.7 GB       | 6.6 GB        | 14 GB       |
| 推論モデル                | ❌            | ✅（Thinking） | ✅（CoT）    |
| 画像入力                  | ❌            | ✅             | ❌           |
| Intelligence Index スコア | 12           | -             | **24**      |
| 最小メモリ                | 〜6 GB       | 〜8 GB        | **16 GB**   |

> Intelligence Index 出典: [Artificial Analysis](https://artificialanalysis.ai/models/gpt-oss-20b) — gpt-oss-20b スコア 24（同サイズクラス平均 15 を大きく上回る）

### なぜスコアが高いのか

- OpenAI の最新 SoTA 学習手法（o3/o4-mini と同様の RL パイプライン）を使用
- MoE 構造で **アクティブパラメータは 3.6B** のため、21B モデルのわりに推論が速い
- ベンチマーク上は OpenAI o3-mini と同等とされる

---

## 動作しない場合の代替手段

### クラウド経由で使う（無料）

動作しない、または非常に遅い場合はクラウド経由で使えます:

```bash
ollama run gpt-oss:20b-cloud
```

`20b-cloud` タグはローカルではなく OpenAI のサーバーで実行されます（インターネット接続必要）。

### OpenAI の Playground で試す

[https://gpt-oss.com/](https://gpt-oss.com/) で直接試せます（アカウント不要）。

---

## トラブルシューティング

| 症状                                | 対処                                                   |
| ----------------------------------- | ------------------------------------------------------ |
| `MXFP4 not supported` エラー        | Ollama を最新版に更新する                              |
| ダウンロードが途中で止まる          | `ollama pull gpt-oss:20b` を再実行（再開される）       |
| メモリ不足でクラッシュ / 起動しない | このPCのRAMが不足している → クラウド版を使う           |
| 応答が非常に遅い                    | `reasoning_effort: low` を設定する                     |
| Continue でモデルが出ない           | `ollama list` でモデル名確認、config.yaml と一致させる |

---

## 参考リンク

- [OpenAI 公式発表 - gpt-oss が登場](https://openai.com/ja-JP/index/introducing-gpt-oss/)
- [Ollama - gpt-oss](https://ollama.com/library/gpt-oss)
- [Artificial Analysis - gpt-oss-20B](https://artificialanalysis.ai/models/gpt-oss-20b)
- [OpenAI Model Card (arxiv)](https://arxiv.org/abs/2508.10925)
