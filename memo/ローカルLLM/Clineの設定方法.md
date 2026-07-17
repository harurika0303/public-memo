# Cline の設定方法（Ollama ローカルモデル接続）

## Cline とは

VS Code 拡張機能。指示を与えると **自律的にファイル作成・編集・ターミナル実行** を行うコーディング Agent。  
Continue（チャット・補完専用）と異なり、エージェントとして動作する。

| 拡張機能  | 主な用途              | ファイル操作         | ターミナル実行 | エージェントループ |
| --------- | --------------------- | -------------------- | -------------- | ------------------ |
| Continue  | チャット・補完        | ❌（手動適用）        | ❌              | ❌                  |
| **Cline** | **コーディングAgent** | ✅ 自律的に作成・編集 | ✅              | ✅                  |

---

## Step 1: インストール

VS Code の拡張機能タブで `Cline` を検索してインストール。

または拡張機能 ID で直接インストール:

```
saoudrizwan.claude-dev
```

---

## Step 2: 初回セットアップ画面

インストール後、左サイドバーの Cline アイコンをクリックすると初回設定画面が開く。

**「Bring my own API key」** を選択する。

> - 「Absolutely Free」→ Cline 側のクラウドサービス（使用制限あり）
> - 「Frontier Model」→ Claude / GPT などの外部 API キーが必要
> - **「Bring my own API key」→ Ollama などの任意プロバイダーを使う（ローカルLLM はこれ）**

---

## Step 3: プロバイダー設定（Configure your provider）

以下のように設定する:

| 設定項目             | 値                       | 備考                                       |
| -------------------- | ------------------------ | ------------------------------------------ |
| API Provider         | `Ollama`                 |                                            |
| Use custom base URL  | ✅ チェックを入れる       |                                            |
| Base URL             | `http://localhost:11434` |                                            |
| Ollama API Key       | （空欄）                 | ローカルインストールでは不要               |
| Model                | `gpt-oss:20b`            | `ollama list` で表示される名前と一致させる |
| Model Context Window | `32768`                  | デフォルトのままでよい                     |
| Request Timeout (ms) | `120000`                 | **デフォルト 30000 は短すぎる → 変更推奨** |

設定後「Save」をクリック。

### ⚠️ 警告について

- **「Does not support Mcp and Focus Chain」**（赤い X）  
  → MCP ツール連携と Focus Chain 機能が使えないという警告。  
  　通常のファイル作成・編集・ターミナル実行には**影響なし**。

- **「Cline uses complex prompts and works best with Claude models」**（オレンジ注記）  
  → Cline のシステムプロンプトが長いため、能力の低いモデルでは誤動作する場合がある。  
  　gpt-oss:20b はある程度動作するが、完璧ではない可能性がある。

---

## Step 4: 動作確認

サイドバーの Cline チャット欄に以下を入力してみる:

```
hello.txt というファイルを現在のフォルダに作成して、「こんにちは！」と書いてください。
```

Cline が:
1. ファイル作成の **差分プレビューを表示** → 「Approve」をクリックして承認
2. ファイルが作成されれば成功

---

## Step 5: 応答が遅い・重い場合の対処

### reasoning_effort を low に設定する

Cline 設定画面（⚙アイコン）→ **Custom Instructions** に以下を入力:

```
reasoning_effort: low
```

### Context Window を小さくする

メモリ不足やクラッシュが起きる場合は **Model Context Window** を `8000` 程度に下げる。

### Request Timeout を増やす

応答途中でタイムアウトエラーになる場合は `Request Timeout` を `180000`（3分）以上に増やす。

---

## 基本的な使い方

### ファイル作成

```
src/utils/format.ts を作成して、日付を yyyy/MM/dd 形式にフォーマットする関数を書いてください
```

### ファイル編集

```
src/index.ts の〇〇関数にエラーハンドリングを追加してください
```

### ターミナル操作込みのタスク

```
React のプロジェクトを新規作成して、npm install まで実行してください
```

Cline は変更前に差分を表示し、ターミナルコマンド実行前にも確認を求める。  
「Approve」で承認、「Reject」で却下できる。

---

## Cline が動作しない場合の代替

**Roo Code**（Cline の派生版）を試す:

```
rooveterinaryinc.roo-cline
```

UI・設定方法はほぼ同じ。Ollama との接続設定も同様の手順でできる。

---

## 関連メモ

- [Ollamaについて](./Ollamaについて.md)
- [GPT-OSS 20B 導入手順](./GPT-OSS_20B_導入手順.md)
