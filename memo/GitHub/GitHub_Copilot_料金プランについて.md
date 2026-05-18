# GitHub Copilot 料金プランについて

> 参考:
> - [Plans for GitHub Copilot - GitHub公式ドキュメント](https://docs.github.com/en/copilot/about-github-copilot/subscription-plans-for-github-copilot)
> - [Usage-based billing for individuals - GitHub公式ドキュメント](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)

---

## ⚠️ 2026年6月1日から料金体系が変わる

**リクエストベース → AIクレジットベース** に移行する。

- 月払いプランは **自動で移行**（操作不要）
- 年払いプランは自動更新されない。GitHubからの通知を確認すること

詳細は後述の「新しい料金体系（AIクレジット）」を参照。

---

## プラン一覧

個人向けプランは以下の4つ。

| プラン           | 月額      | 対象                                   |
| ---------------- | --------- | -------------------------------------- |
| **Copilot Free** | 無料      | すべてのGitHubユーザー                 |
| **Copilot Pro**  | $10 / 月  | 個人開発者                             |
| **Copilot Pro+** | $39 / 月  | ヘビーユーザー・上位モデルを使いたい人 |
| **Copilot Max**  | $100 / 月 | 最大限使いたい人                       |

チーム・企業向け（参考）:
- **Copilot Business**: $19 / シート / 月
- **Copilot Enterprise**: $39 / シート / 月

---

## 各プランでできること（個人向け）

### コード補完・インライン補完

|                       | Free          | Pro        | Pro+       |
| --------------------- | ------------- | ---------- | ---------- |
| コード補完            | 月2,000回まで | **無制限** | **無制限** |
| Next Edit Suggestions | ○             | ○          | ○          |

> コード補完はAIクレジットを消費しない。Tab で補完を受け入れる機能は有料プランなら無制限。

### Copilot Chat（チャット）

|                    | Free           | Pro        | Pro+       |
| ------------------ | -------------- | ---------- | ---------- |
| IDEでのチャット    | 月50メッセージ | **無制限** | **無制限** |
| インラインチャット | ○              | ○          | ○          |
| スラッシュコマンド | ○              | ○          | ○          |

### 使えるAIモデル（抜粋）

| モデル                  | Free | Pro | Pro+ |
| ----------------------- | ---- | --- | ---- |
| GPT-4.1                 | ○    | ○   | ○    |
| GPT-5 mini              | ○    | ○   | ○    |
| Claude Haiku 4.5        | ○    | ○   | ○    |
| Gemini 2.5 Pro          | -    | ○   | ○    |
| Claude Sonnet 4.5 / 4.6 | -    | -   | ○    |
| Claude Opus 4.7         | -    | -   | ○    |

> 高性能なモデルはPro以上のプランでないと使えない。

### その他の機能

|                     | Free | Pro | Pro+ |
| ------------------- | ---- | --- | ---- |
| Copilot cloud agent | -    | ○   | ○    |
| Agent mode（IDE）   | ○    | ○   | ○    |
| PRサマリー自動生成  | -    | ○   | ○    |
| Copilot CLI         | ○    | ○   | ○    |
| カスタム指示        | ○    | ○   | ○    |

---

## 新しい料金体系（2026年6月〜）：AIクレジットとは

### AIクレジットの基本

- **1 AI credit = $0.01 USD**
- Copilotを使う際にトークンを消費し、その量に応じてAIクレジットが引かれる
- 毎月一定のAIクレジットがプランに含まれる

### 各プランの月間AIクレジット量

| プラン       | 基本クレジット | フレックス | 合計       |
| ------------ | -------------- | ---------- | ---------- |
| Copilot Free | 少量           | -          | 少量       |
| Copilot Pro  | 1,000          | 500        | **1,500**  |
| Copilot Pro+ | 3,900          | 3,100      | **7,000**  |
| Copilot Max  | 10,000         | 10,000     | **20,000** |

> **基本クレジット**は固定。**フレックス**はAIの価格変動に応じて変わる可変分。

### 上限を超えた場合

- 追加の予算（$単位）を設定すれば引き続き使える
- $10の予算 = 1,000 AIクレジット分
- 上限を超えたくない場合は次の月の更新まで待てばよい

### AIクレジットを消費する機能・しない機能

| 機能                     | AIクレジット消費     |
| ------------------------ | -------------------- |
| コード補完（インライン） | **消費しない**       |
| Next Edit Suggestions    | **消費しない**       |
| Copilot Chat             | **消費する**         |
| Agent mode               | **消費する**（多め） |
| Copilot cloud agent      | **消費する**（多め） |
| Copilot CLI              | **消費する**         |

---

## 参考

- [Plans for GitHub Copilot](https://docs.github.com/en/copilot/get-started/plans)
- [Usage-based billing for individuals](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-individuals)
- [Preparing for your move to usage-based billing](https://docs.github.com/en/copilot/how-tos/manage-and-track-spending/prepare-for-your-move-to-usage-based-billing)
