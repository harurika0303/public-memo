# GitHub Copilot CLI 入門

> **作成日**: 2026年5月24日  
> **対象**: GitHub Copilot Pro 以上（全プランで利用可）  
> **関連ファイル**: [コード補完とNESの活用法](./GitHub_Copilot_コード補完とNESの低コスト活用法.md)

---

## 0. Copilot CLI とは何か

**ターミナル上でAIエージェントと会話しながらコーディング作業を丸ごとこなせるツール。**

VS Code の Copilot が「エディタの中の補助」なのに対し、Copilot CLI は「ターミナルから自律的にファイルを読み書きし、コマンドを実行し、GitHub.com と連携できるエージェント」。

```
できること:
- ファイルの作成・編集・削除
- シェルコマンドの実行（git / テスト / ビルド 等）
- GitHub の Issue・PR の作成・確認・マージ
- コードベース全体の調査・説明
- 深いリサーチ（Web 検索 + GitHub Code Search）
```

### 「2種類の Copilot CLI」を混同しないこと

| ツール                                     | インストール                             | 用途                             |
| ------------------------------------------ | ---------------------------------------- | -------------------------------- |
| **`gh copilot` 拡張**（旧）                | `gh extension install github/gh-copilot` | コマンドの説明・提案（シンプル） |
| **`copilot` コマンド**（新・本稿のメイン） | `npm install -g @github/copilot` 等      | 本格的なエージェント作業         |

本稿では新しい **スタンドアロン `copilot` コマンド**を解説する。

---

## 1. インストール方法

### 方法① npm（全OS対応・推奨）

```bash
# Node.js 22 以上が必要
npm install -g @github/copilot

# インストール確認
copilot version
```

### 方法② WinGet（Windows）

```powershell
winget install GitHub.Copilot
```

### 方法③ Homebrew（macOS / Linux）

```bash
brew install copilot-cli
```

### 方法④ インストールスクリプト（macOS / Linux）

```bash
curl -fsSL https://gh.io/copilot-install | bash
```

---

## 2. 認証（初回のみ）

### ブラウザ認証（標準）

```bash
copilot login
```

ブラウザが開き、GitHub の OAuth 認証を完了するとトークンがシステムの認証情報ストアに保存される。

### 環境変数で認証（CI・自動化向け）

```bash
# 優先度: COPILOT_GITHUB_TOKEN > GH_TOKEN > GITHUB_TOKEN
export COPILOT_GITHUB_TOKEN=github_pat_xxx...
```

Fine-grained PAT を使う場合は「Copilot Requests」権限を付与して発行する。

---

## 3. 起動と基本的な使い方

### インタラクティブモードで起動（基本）

```bash
# プロジェクトのディレクトリで起動
cd ~/my-project
copilot
```

起動するとチャット形式のインターフェースが開く。あとは日本語で話しかけるだけ。

**ローカルの作業例:**

```
> H1ヘッダーの背景色をダークブルーに変えてください

> このリポジトリの変更をコミットしてください

> CHANGELOG.md の直近5件の変更を、誰が・いつ・何を変えたかまとめて

> src/content.js の改善点を提案してください

> このプロジェクトのREADMEを初心者向けに書き直してください
```

**GitHub.com との連携例:**

```
> 自分のオープンなPRを一覧表示して

> https://github.com/myorg/myrepo/issues/42 に取り組んで、適切な名前のブランチを作って

> このリポジトリのルートに user-info.js を追加してPRを作って

> PR #57 の変更を確認して、重大なエラーがあれば教えて
```

### プログラマティックモード（1回だけ実行して終了）

```bash
# -p でプロンプトを渡す、--allow-all-tools で確認なしに実行
copilot -p "今週のコミットを一覧表示してまとめて" --allow-tool='shell(git)'

# 完全自動（確認なし）- CI 等での使用
copilot -p "テストを実行してエラーを修正して" --allow-all
```

---

## 4. 3つの動作モード

`Shift + Tab` で切り替えられる。

| モード                  | 説明                             | 使いどころ             |
| ----------------------- | -------------------------------- | ---------------------- |
| **通常（ask/execute）** | 質問・実行を繰り返す対話形式     | 普段の作業全般         |
| **Plan モード**         | コードを書く前に実装計画を立てる | 複雑・大規模なタスク   |
| **Autopilot モード**    | 確認なしに自律実行し続ける       | 単純な反復作業・自動化 |

Plan モードでは「何をどの順で変更するか」を提案してから着手するので、大きな変更のときに誤りを事前に防げる。

---

## 5. よく使うスラッシュコマンド

インタラクティブモード中に `/` で始めるコマンド。

### コンテキスト・セッション管理

| コマンド        | 説明                                                      |
| --------------- | --------------------------------------------------------- |
| `/init`         | このリポジトリの `copilot-instructions.md` を自動生成する |
| `/context`      | トークン使用量を表示                                      |
| `/compact`      | 会話履歴を圧縮してコンテキストを節約                      |
| `/clear`        | 新しい会話を開始                                          |
| `/undo`         | 最後のターンを巻き戻し、ファイル変更も元に戻す            |
| `/resume`       | 過去のセッションを再開                                    |
| `/instructions` | カスタム指示ファイルの確認・切り替え                      |

### コード・開発作業

| コマンド                      | 説明                                       |
| ----------------------------- | ------------------------------------------ |
| `/diff`                       | 現在のディレクトリの変更を確認             |
| `/pr view\|create\|fix\|auto` | PRの確認・作成・修正                       |
| `/review`                     | コードレビューエージェントを実行           |
| `/plan`                       | 実装計画を先に立てる                       |
| `/fleet`                      | タスクを並列で実行（サブエージェント分散） |
| `/research TOPIC`             | 深いリサーチ（GitHub + Web 検索）          |

### 設定・ツール管理

| コマンド             | 説明                               |
| -------------------- | ---------------------------------- |
| `/model`             | 使用するAIモデルを変更             |
| `/mcp`               | MCP サーバーを管理                 |
| `/skills`            | スキルを管理                       |
| `/agent`             | 利用可能なエージェントを選ぶ       |
| `/allow-all` `/yolo` | 全ての操作を確認なしに許可（注意） |
| `/feedback`          | フィードバックを送る               |

---

## 6. 便利なショートカット（インタラクティブモード）

| 操作                                      | キー           |
| ----------------------------------------- | -------------- |
| ファイルをコンテキストに追加              | `@ファイル名`  |
| Issue / PR をコンテキストに追加           | `#番号`        |
| Copilotをスキップしてシェルコマンドを実行 | `!コマンド`    |
| 次のモードに切り替え                      | `Shift + Tab`  |
| 処理中に追加メッセージを送る              | `Ctrl + Enter` |
| 処理をバックグラウンドに移す              | `Ctrl + X → b` |
| ヘルプを表示                              | `?`            |
| 終了                                      | `Ctrl + D`     |
| 画面クリア                                | `Ctrl + L`     |

### `@ファイル名` の使い方例

```
> @src/App.tsx このファイルのパフォーマンスを改善して

> @docs/spec.md この仕様に沿ってAPIを実装して

> @README.md この内容をもとに CONTRIBUTING.md を作って
```

---

## 7. ツールの権限管理

Copilot はファイル変更・コマンド実行のたびに確認を求めてくる。

### インタラクティブ時の応答

| キー | 意味                         |
| ---- | ---------------------------- |
| `y`  | 今回だけ許可                 |
| `!`  | このセッション中は常に許可   |
| `n`  | 拒否（別の方法を伝えられる） |
| `#`  | このセッション中は常に拒否   |

### 起動時オプションで制御

```bash
# 全ての操作を自動許可（自動化・信頼できる環境のみ）
copilot --allow-all

# git コマンドだけ自動許可
copilot --allow-tool='shell(git)'

# git push だけ禁止、他は全許可
copilot --allow-all-tools --deny-tool='shell(git push)'

# ファイル書き込みだけ自動許可
copilot --allow-tool='write'
```

> ⚠️ `--allow-all` は信頼できるプロジェクトディレクトリ内でのみ使用すること。ホームディレクトリ（`~`）では絶対に使わない。

---

## 8. モデルの変更（BYOK）

### 使用モデルをセッション内で変更

```
/model
```

一覧から選択する。

### BYOK（自分のAPIキーを使う）

```bash
# Anthropic を使う例
export COPILOT_PROVIDER_TYPE=anthropic
export COPILOT_PROVIDER_BASE_URL=https://api.anthropic.com
export COPILOT_PROVIDER_API_KEY=sk-ant-xxxxx
export COPILOT_MODEL=claude-opus-4-7

copilot
```

```bash
# Ollama（ローカルモデル）を使う例
export COPILOT_PROVIDER_TYPE=openai       # Ollama は OpenAI 互換
export COPILOT_PROVIDER_BASE_URL=http://localhost:11434/v1
export COPILOT_MODEL=llama3.1

copilot
```

BYOK で使うモデルは **tool calling（関数呼び出し）とストリーミングの両対応が必須**。コンテキストウィンドウは 128k tokens 以上推奨。

---

## 9. カスタマイズ

### 9-1. `copilot-instructions.md`（Chat/Agent への常時指示）

プロジェクトの技術スタック・規約・テスト方法などを書いておくと、毎回説明しなくて済む。

```bash
# 自動生成（リポジトリを分析して作成）
copilot
> /init
```

または手動で `.github/copilot-instructions.md` を作成する。

```markdown
## プロジェクト概要
Laravelベースの在庫管理API。

## ビルド・テスト
- 依存関係: composer install
- DB セットアップ: php artisan migrate --seed
- テスト: php artisan test

## コーディング規約
- バリデーションは FormRequest クラスで行う
- APIレスポンスは ApiResource を通す
```

### 9-2. Agent Skills（専門ワークフロー）

よく使う手順をスキルとして定義すると、「それっぽい依頼」をしたときに自動ロードされる。

```
.github/skills/
└── laravel-test/
    └── SKILL.md
```

```markdown
---
name: laravel-test
description: Laravelのテスト実行とカバレッジ確認。テスト関連の依頼に使用。
---

## テスト実行手順
1. `php artisan test` で全テスト実行
2. 失敗があれば `php artisan test --filter=失敗したテスト名` で絞り込み
3. カバレッジ: `php artisan test --coverage`
```

### 9-3. MCP サーバー（外部ツール連携）

```
# インタラクティブモードで追加
/mcp add

# コマンドラインで追加（stdio 形式）
copilot mcp add my-db-server -- npx my-db-mcp-server
```

**ビルトインMCPサーバー（追加設定不要）:**

| サーバー名          | 機能                                          |
| ------------------- | --------------------------------------------- |
| `github-mcp-server` | GitHub API（Issue・PR・Code Search・Actions） |
| `playwright`        | ブラウザ自動操作（クリック・スクショ等）      |
| `fetch`             | HTTP リクエスト                               |
| `time`              | 現在時刻・タイムゾーン変換                    |

### 9-4. カスタムエージェント

特定の役割に特化したエージェントを定義できる。

```
.github/agents/
└── frontend-expert.agent.md
```

```markdown
---
description: フロントエンド専門エージェント。React/TypeScript/CSS の設計・実装に特化。
model: claude-sonnet-4-6
---

あなたはフロントエンドの専門家です。
React の関数コンポーネントと TypeScript を使った実装を行います。
アクセシビリティ（WCAG 2.1 AA）を常に意識してください。
```

**ビルトインエージェント（最初から使える）:**

| エージェント名    | モデル            | 用途                                   |
| ----------------- | ----------------- | -------------------------------------- |
| `code-review`     | Claude Sonnet 4.5 | バグ・セキュリティ・ロジックのレビュー |
| `explore`         | Claude Haiku 4.5  | 高速なコードベース調査（並列実行可）   |
| `general-purpose` | Claude Sonnet 4.5 | 複雑な多段階タスク                     |
| `research`        | Claude Sonnet 4.6 | コード・リポジトリ・Webのリサーチ      |
| `rubber-duck`     | 補完モデル        | 設計・実装の批判的レビュー             |
| `task`            | Claude Haiku 4.5  | ビルド・テスト・Lint の実行            |

```
# エージェントを指定して使う例
copilot --agent=code-review
> @src/auth.py このファイルのセキュリティ問題を確認して
```

---

## 10. 実践的なユースケース例

### Issue に取り組む

```
> https://github.com/myorg/myrepo/issues/42 に取り組んでください。
  適切な名前のブランチを作成し、変更後にPRを作ってください。
```

### テストを書いてもらう

```
> src/services/UserService.php の単体テストを書いて。
  既存のテスト（tests/Unit/）のスタイルに合わせてください。
```

### バグを調査・修正する

```
> php artisan test を実行すると UserControllerTest が失敗します。
  原因を調査して修正してください。
```

### PRのコードレビュー

```
> https://github.com/myorg/myrepo/pull/99 の変更を確認して、
  重大な問題があればコメントしてください。
```

### GitHub Actions ワークフローを作る

```
> main ブランチへのPR時に ESLint を実行する GitHub Actions ワークフローを
  作成してください。エラーがある場合はPRのチェックを失敗させてください。
  変更をブランチに push して PR を作ってください。
```

### プロジェクトをゼロから作る

```
> create-next-app と Tailwind CSS を使って Next.js アプリを作ってください。
  GitHub API からこのリポジトリのビルド成功率・平均ビルド時間・テスト合格率を
  表示するダッシュボードにしてください。
  完成後、ブラウザで確認するための手順を教えてください。
```

---

## 11. `/fleet`（並列実行）を使いこなす

複数のサブエージェントが並列でタスクを実行できる機能。

```
> /fleet 以下のタスクを並列で実行してください:
  1. src/models/ の全ファイルに JSDoc を追加
  2. src/controllers/ の全ファイルの TODO を確認してまとめる
  3. テストカバレッジが50%以下のファイルを一覧表示
```

---

## 12. コスト（クレジット消費）

- Copilot CLI への**プロンプト1回 = クレジット消費1回**（モデルの倍率に依存）
- 例: `Claude Sonnet 4.5 (1x)` → 1回のプロンプトで1クレジット消費
- **コード補完・NES とは異なり、全て消費対象**
- Pro ($10/月) では使いすぎに注意。大量自動化には Pro+ ($39/月) が向いている

---

## 情報源

| 内容                                                                         | URL                                                                                           | 参照日     |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ---------- |
| GitHub Copilot CLI の概要                                                    | https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli                          | 2026-05-24 |
| Copilot CLI のインストール方法                                               | https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli | 2026-05-24 |
| Copilot CLI コマンドリファレンス（スラッシュコマンド・ショートカット全一覧） | https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference      | 2026-05-24 |
| Copilot CLI の責任ある使い方                                                 | https://docs.github.com/en/copilot/responsible-use/copilot-cli                                | 2026-05-24 |
| Copilot CLI のセットアップ全体                                               | https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli                     | 2026-05-24 |
