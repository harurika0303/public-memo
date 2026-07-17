# GitHub Copilot コード補完・NES と Chat/Agent の使い分けと低コスト活用法

> **作成日**: 2026年5月24日  
> **関連ファイル**: [AIコーディング環境のコスパ比較と最適構成](../ローカルLLM/AIコーディング環境のコスパ比較と最適構成.md)

---

## 前提：機能ごとのクレジット消費と「何が効くか」

GitHub Copilot の機能は大きく2種類あり、**カスタマイズ指示が効くかどうか**も異なる。

| 機能                             | クレジット消費     | `copilot-instructions.md` | `instructions/*.md` | Agent Skills |
| -------------------------------- | ------------------ | ------------------------- | ------------------- | ------------ |
| **コード補完（Ghost Text）**     | **なし（無制限）** | ❌ 効かない                | ❌ 効かない          | ❌ 効かない   |
| **Next Edit Suggestions（NES）** | **なし（無制限）** | ❌ 効かない                | ❌ 効かない          | ❌ 効かない   |
| **Copilot Chat**                 | あり               | ✅ 効く                    | ✅ 効く              | ✅ 効く       |
| **エージェントモード**           | あり               | ✅ 効く                    | ✅ 効く              | ✅ 効く       |

**コード補完と NES は、指示ファイルの内容を一切参照しない。** 代わりに VS Code で開いているファイルのコンテキストを使って提案する。

**→ コスパ戦略：** 補完・NES を主力にし、Chat・Agent は必要なときだけ使う。

---

## 1. コード補完（Ghost Text）の精度を上げる方法

補完の精度は「開いているファイルの内容」で決まる。`copilot-instructions.md` は無関係。

### 1-1. 関連ファイルを VS Code のタブで開いておく

Copilot はアクティブなエディタと**現在開いているタブのファイル**をコンテキストとして参照する。

```
// 効果的な準備の例（Laravelプロジェクト）
- 実装する Controller を開く
- 参照する Model ファイルを別タブで開いておく
- docs/db-schema.md や docs/api-design.md を開いておく
  → Copilot がスキーマ定義を「見た上で」補完する
```

### 1-2. コード内にコメントで意図を書く

```php
// ユーザーIDから注文履歴を取得し、金額の降順で返す
public function getOrderHistory(int $userId): Collection
{
    // ↑ このコメントだけで、以下の実装が補完される
```

### 1-3. 型アノテーションを先に書く

```typescript
// 引数・戻り値の型が明確だと補完精度が大幅に上がる
function calculateTotalPrice(items: CartItem[]): number {
    // ← ここを空にしておくと実装を補完してくれる
}
```

### 1-4. 部分的な名前から書き始める

命名の一部を書くだけで残りを補完する。`getUserOrd` と書けば `getUserOrderHistory` を提案する。

---

## 2. Next Edit Suggestions（NES）

### 概要

**「今の編集の流れから、次に必要な場所と変更内容を先読みする」機能。**  
カーソル位置での補完（Ghost Text）と違い、**ファイル内の別の場所への変更も提案できる**。

> Ghost text suggestions are great at autocompleting a section of code. But since most coding activity is editing existing code, it's a natural evolution of inline suggestions to also help with edits.  
> — VS Code 公式ドキュメント

NES も Ghost Text と同様、`copilot-instructions.md` は参照しない。開いているファイルの内容だけを見る。

### 2-1. 操作方法

| 操作                     | キー  |
| ------------------------ | ----- |
| 次の提案箇所へジャンプ   | `Tab` |
| 提案を承認（ジャンプ後） | `Tab` |
| 提案を却下               | `Esc` |

行番号の左側（ガター）に矢印アイコンが表示され、次の提案がどの方向にあるかを示す。

### 2-2. 主なユースケース

#### タイポの修正

```
conts x = 5;   → const x = 5;  を提案
cont  x = 5;   → const x = 5;  を提案
```

#### 論理ミスの修正

```typescript
// || を && に直すよう提案するケース
if (isAdmin || hasPermission) { ... }
```

#### クラス変更の波及

```typescript
// Point → Point3D に変えたとき
class Point3D {
    x: number;
    y: number;
    // ↑ 編集後、NES が順次提案:
    //   1. z: number; を追加
    //   2. distance() メソッドの計算式に z を追加
}
```

#### 変数名・関数名のリネーム

```
// userId を customerId に1箇所変えると、
// 同じファイル内の他の userId 使用箇所に変更を提案
```

#### コピペ後のスタイル合わせ

```
// 別ファイルからペーストしたコードを
// 現在のファイルのスタイルに合わせるよう提案
```

### 2-3. 有効化（VS Code）

```json
{
  "github.copilot.nextEditSuggestions.enabled": true,
  "github.copilot.nextEditSuggestions.fixes": true
}
```

`fixes: true` にすると、不足インポートなど診断（スクワイグル）ベースの提案も有効になる。

### 2-4. NES に使われるモデルを変更する

**NES 専用のモデル設定は存在しない。** NES は Ghost Text と同じ「インライン補完モデル」を共有しており、コード補完のモデルを変更すると NES にも反映される。

```
変更方法①: コマンドパレット
  F1 → "GitHub Copilot: Change Completions Model" → 選択

変更方法②: メニュー
  タイトルバー Chat メニュー
  → "Configure Inline Suggestions..."
  → "Change Completions Model..."
```

**注意事項：**
- 選択肢が1つしか表示されないこともある（モデルは随時追加予定）
- **Pro+（$39/月）の方が選べるモデルの幅が広い**
- Copilot Business / Enterprise は管理者が `Editor Preview Features` を有効にする必要あり
- **ローカルモデルをインライン補完に使うことは現時点では不可**（Chat のみ対応）

---

## 3. Chat・Agent モード向けのカスタマイズ（補完・NES には効かない）

以下の仕組みは **Chat とエージェントモードにのみ効く**。補完・NES には無関係。

### 3-1. `copilot-instructions.md`（リポジトリ全体の常時指示）

`.github/copilot-instructions.md` を作成すると、そのリポジトリ内の Chat・Agent リクエストに自動付与される。

```markdown
## プロジェクト概要
これはLaravelベースの在庫管理アプリです。

## コーディング規約
- バリデーションは必ず FormRequest クラスで行う
- エラーレスポンスは { "message": "...", "errors": {...} } 形式
- Eloquentモデルには必ず $fillable を定義する

## ビルド・テスト
- composer install → php artisan migrate → php artisan db:seed
- テスト実行: php artisan test
```

VS Code の Chat で `/init` と入力すると、現在のリポジトリを分析して `copilot-instructions.md` を自動生成してくれる。

### 3-2. パス別指示ファイル（`.github/instructions/*.instructions.md`）

`applyTo` にグロブを指定すると、一致するファイルに関わる Chat・Agent リクエストにのみ適用される。

```markdown
---
applyTo: "app/Models/**/*.php"
---
Eloquentモデルには必ず $fillable を定義する。
リレーションは hasMany / belongsTo 等を明示的に定義する。
```

```markdown
---
applyTo: "**/*.ts,**/*.tsx"
---
React コンポーネントは関数コンポーネントで書く。
型定義は interface より type を優先する。
```

### 3-3. Agent Skills（`.github/skills/SKILL.md`）

`copilot-instructions.md` がテキスト指示のみなのに対し、Skills はスクリプト・例・リソースを含む「専門ワークフロー」を定義できる。エージェントが関連性を判断して自動ロードするか、`/スキル名` で手動呼び出す。

```
.github/skills/
└── laravel-artisan/          ← ディレクトリ名 = スキル名
    └── SKILL.md
```

```markdown
---
name: laravel-artisan
description: Laravelのartisanコマンド実行・マイグレーション・シーダー操作。artisanやLaravelタスクの依頼に使用。
---

## よく使うコマンド
- `php artisan migrate` でマイグレーション実行
- `php artisan db:seed` でシーダー実行
- `php artisan make:model Xxx -mcr` でModel/Migration/Controller一括生成
```

**`copilot-instructions.md` との違い：**

|                | `copilot-instructions.md` | Agent Skills                       |
| -------------- | ------------------------- | ---------------------------------- |
| 適用タイミング | 常時・自動                | 関連検出時 or `/コマンド`          |
| 内容           | テキスト指示のみ          | 指示 + スクリプト + リソース       |
| ポータビリティ | VS Code・GitHub のみ      | VS Code・CLI・クラウドエージェント |

### 3-4. カスタマイズファイルの全体像

```
.github/
├── copilot-instructions.md        ← Chat/Agent に常時適用
├── instructions/
│   ├── models.instructions.md     ← パス別（applyTo で指定）
│   └── api.instructions.md
├── prompts/
│   └── review.prompt.md           ← 繰り返しタスク（手動呼び出し）
└── skills/
    └── laravel-artisan/
        └── SKILL.md               ← 専門ワークフロー（自動 or /コマンド）
```

---

## 4. 機能の使い分けフロー

```
新しいメソッドを実装したい
  → 関連ファイルを開く + コメントで意図を書く → Ghost Text 補完（無制限）

変数名をリネームしたい・ミスを直したい
  → NES の Tab ナビゲート（無制限）

「この設計どう思う？」「テストコードを書いて」
  → Copilot Chat（クレジット消費）
    ※ copilot-instructions.md の内容が考慮される

複数ファイルにまたがる大規模変更
  → エージェントモード（クレジット消費）
    ※ copilot-instructions.md + Skills が考慮される
```

---

## 5. ベストプラクティスリポジトリ・参考資料

| 内容                                                                         | URL                                                                                    |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| GitHub 公式コミュニティコレクション（Skills・Agents・Instructions・Prompts） | https://github.com/github/awesome-copilot                                              |
| GitHub Docs: Custom instructions ライブラリ（**日本語版あり**）              | https://docs.github.com/ja/copilot/tutorials/customization-library/custom-instructions |
| Anthropic 公式リファレンス Skills 集                                         | https://github.com/anthropics/skills                                                   |
| Agent Skills 公開標準仕様                                                    | https://agentskills.io/                                                                |

GitHub Docs のライブラリには「コードレビュアー」「デバッグチューター」「GitHub Actions ヘルパー」「テスト自動化」など用途別のテンプレートが日本語で揃っている。

---

## 情報源

| 内容                                                  | URL                                                                                                  | 参照日     |
| ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ---------- |
| VS Code 公式: インライン補完・NES の説明              | https://code.visualstudio.com/docs/copilot/ai-powered-suggestions                                    | 2026-05-24 |
| VS Code 公式: AI 言語モデルの変更方法                 | https://code.visualstudio.com/docs/copilot/customization/language-models                             | 2026-05-24 |
| VS Code 公式: Agent Skills の使い方                   | https://code.visualstudio.com/docs/copilot/customization/agent-skills                                | 2026-05-24 |
| VS Code 公式: Copilot カスタマイズ概要                | https://code.visualstudio.com/docs/copilot/copilot-customization                                     | 2026-05-24 |
| GitHub 公式: カスタム指示ファイルの追加方法           | https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot | 2026-05-24 |
| GitHub Blog: Usage-based billing への移行（6月1日〜） | https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/      | 2026-05-24 |
