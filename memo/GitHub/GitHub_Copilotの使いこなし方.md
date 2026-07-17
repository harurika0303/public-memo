# GitHub Copilot の使いこなし方

> 参考:
> - [What is GitHub Copilot? - GitHub公式ドキュメント](https://docs.github.com/en/copilot/about-github-copilot/what-is-github-copilot)
> - [GitHub Copilot features - GitHub公式ドキュメント](https://docs.github.com/en/copilot/about-github-copilot/github-copilot-features)

---

## GitHub Copilot とは

AIコーディングアシスタント。コードを書く速度を上げ、問題解決や協業に集中できるようにすることを目的としている。

**使える場所**:
- IDE（VS Code など）
- GitHub Web サイト
- GitHub Mobile
- コマンドライン（GitHub CLI）
- Windows Terminal

---

## 機能一覧

公式では機能を3つのカテゴリに分類している。

---

### 1. アシスティブ機能（Assistive features）

作業中にリアルタイムで提案・サポートしてくれる機能。

#### インライン補完（Inline suggestions）

コードを書きながら自動補完候補を表示する。いわゆる「コード補完」。

- `Tab` で候補を受け入れ、`Esc` で却下
- VS Code では **Next Edit Suggestions** も使える
  - 次に編集しそうな箇所を予測して補完してくれる

#### Copilot Chat

コーディングに関する質問をチャット形式でやりとりできる。

- IDE のサイドパネル、GitHub Web、GitHub Mobile で使用可能
- コードの説明・バグの原因分析・リファクタリング提案などに使える

**使い方の例（VS Code）**:
- `Ctrl + Alt + I` でチャットパネルを開く
- コードを選択して右クリック → `Copilot` でインラインチャットも使える

#### Copilot in GitHub Desktop

コミット時にメッセージと説明文を自動生成してくれる。

#### Copilot プルリクエストサマリー

PRで変更された内容・影響ファイル・レビュアーが注目すべき点を自動で要約してくれる。

---

### 2. エージェント機能（Agentic features）

人間の監督なしに自律的に動作できる機能。ただし、コマンド実行やPRのマージなど重要な操作は人間の承認が必要。

#### エージェントモード（Agent mode in IDEs）

IDEで自律的に動作するモード。

- 変更すべきファイルを自分で判断する
- コード変更とターミナルコマンドを提案し、承認を求める
- 問題が発生したら自分で修正を繰り返す

**VS Code での使い方**:
- チャット入力欄のモード切り替えで `Agent` を選択

#### Copilot コードレビュー（Copilot code review）

AIがコードレビューの提案を行う。コードの品質向上に使える。

#### Copilot CLI

ターミナル上でCopilotを使う機能（GitHub CLI 経由）。

```bash
# インストール
gh extension install github/gh-copilot

# 使い方例
gh copilot suggest "dockerコンテナを削除するコマンド"
gh copilot explain "git rebase -i HEAD~3"
```

#### Copilot クラウドエージェント（Copilot cloud agent）

リポジトリを調査し、実装計画を立て、ブランチにコード変更を加える自律エージェント。
GitHubのIssueをCopilotにアサインすることも可能。
※ Pro+ / Business / Enterprise のみ利用可能

---

### 3. カスタマイズ機能（Features for customization）

Copilot に与えるコンテキストを充実させ、回答の質を上げる機能。

#### カスタム指示（Custom instructions）

自分の好みやプロジェクトのルールをCopilotに伝えることができる。

- `.github/copilot-instructions.md` をリポジトリに置くと自動で読み込まれる
- 「このプロジェクトはLaravelを使っている」「コメントは日本語で書く」などを設定できる

#### プロンプトファイル（Prompt files）

再利用可能なプロンプトをMarkdownファイルとして保存・共有できる。
`.github/prompts/` フォルダに `.prompt.md` ファイルを置く。

#### Copilot Spaces

コード・ドキュメント・仕様書などを「Space」にまとめて整理し、Copilotの回答に反映させる機能。

#### MCP サーバー（MCP servers）

Model Context Protocol（MCP）サーバーを設定することで、外部ツールやデータソースにCopilotがアクセスできるようになる。

---

## よく使うショートカット（VS Code）

| 操作                 | ショートカット   |
| -------------------- | ---------------- |
| チャットパネルを開く | `Ctrl + Alt + I` |
| インラインチャット   | `Ctrl + I`       |
| 補完候補を受け入れ   | `Tab`            |
| 補完候補を却下       | `Esc`            |
| 次の候補を表示       | `Alt + ]`        |
| 前の候補を表示       | `Alt + [`        |

---

## 使いこなしのヒント

- **コンテキストを与えるほど精度が上がる** — 質問する前にコードを選択しておく、ファイルを開いておくなど
- **チャットで「〜してください」と具体的に頼む** — 曖昧な質問より具体的な指示の方が良い回答が得られる
- **インライン補完は途中まで書いてから待つ** — 関数名やコメントを書くだけで続きを提案してくれる
- **カスタム指示を設定しておく** — プロジェクトのルールや使用技術を伝えておくと毎回説明しなくて済む
