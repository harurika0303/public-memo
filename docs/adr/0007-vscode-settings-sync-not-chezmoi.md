# ADR-0007: VSCode の設定は Settings Sync で管理し chezmoi のスコープ外とする

## ステータス
承認済み

## 背景
chezmoi で管理するスコープの一つとして VSCode の `settings.json` が候補に挙がった。
ただし WSL 開発環境における VS Code の設定管理には以下の構造的特徴がある：

- 実体は **Windows 側**の `%APPDATA%\Code\User\settings.json` に存在する
- WSL Remote 側は Windows 側の設定を継承するため、別途 settings.json を持たない
- chezmoi は WSL（Linux）上のホームディレクトリを管理対象とするため、Windows 側ファイルの管理には OS テンプレートによる複雑な分岐が必要になる

また将来 Mac を使う可能性があり、OS 間で同一の設定を保ちたいという要求も存在した。

## 決定
VSCode の設定は **VS Code 内蔵の Settings Sync（GitHub アカウント連携）** で管理し、chezmoi の管理対象には含めない。

- settings.json・keybindings.json・拡張機能リストはすべて Settings Sync が担当する
- chezmoi の管理ファイルリストから VSCode 関連を除外する
- WSL・Windows・Mac 間の設定統一は Settings Sync が自動で行う

## 代替案

| 案                                     | 内容                                       | 採用しなかった理由                                                             |
| -------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------ |
| chezmoi の OS テンプレートで管理       | `{{ if eq .chezmoi.os "windows" }}` で分岐 | chezmoi は WSL（Linux）上で動作するため Windows 側パスへのアクセスが複雑になる |
| WSL 内 settings.json のみ chezmoi 管理 | `~/.vscode-server/` 配下を管理             | WSL Remote 側は Windows 側設定を継承するため、独立した管理ファイルが生まれない |

## 結果

- chezmoi リポジトリの管理対象は `~/.zshrc` / `~/.gitconfig` / `~/.ssh/config` に絞られ、シンプルになる
- VS Code の設定変更はすべて Settings Sync 経由で全マシンに自動反映される
- Mac 移行時も chezmoi の変更なしで VSCode 設定が引き継がれる
