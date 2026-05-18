# WSL再インストール後のGitLab SSH接続エラー対処法

## 背景

WSLが壊れてUbuntuを再インストールした後、GitLabにSSHキーを登録したが `Permission denied` エラーが発生した。

---

## よくある原因と対処法

### 1. SSHキーのパーミッションが正しくない

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa        # 秘密鍵
chmod 644 ~/.ssh/id_rsa.pub    # 公開鍵
```

### 2. SSHエージェントに鍵が登録されていない

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519    # ED25519の場合
# ssh-add ~/.ssh/id_rsa     # RSAの場合
```

### 3. GitLabに登録した公開鍵と秘密鍵が一致していない

再インストール後に新しく鍵を生成した場合、**古い公開鍵**がGitLabに残っている可能性がある。

```bash
# 現在の公開鍵を確認
cat ~/.ssh/id_ed25519.pub    # ED25519の場合
# cat ~/.ssh/id_rsa.pub     # RSAの場合
```

GitLabの **アバター > Edit profile > Access > SSH keys** で登録済みの鍵と一致しているか確認してください。

### 4. `~/.ssh/config` が未設定

```
# ~/.ssh/config
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519    # ED25519の場合
  # IdentityFile ~/.ssh/id_rsa      # RSAの場合
```

### 5. SSHキー自体が生成されていない

```bash
ls ~/.ssh/
# id_ed25519 と id_ed25519.pub が存在するか確認（ED25519の場合）
# id_rsa と id_rsa.pub が存在するか確認（RSAの場合）
```

---

## 接続テスト（原因の絞り込み）

```bash
ssh -vT git@gitlab.com
```

`-v` オプションで詳細ログが出力され、どのステップで失敗しているか確認できる。

---

## SSHキーの再生成手順（一から作り直す場合）

公式推奨は **ED25519**（RSAより安全・高速）。

```bash
# 1. 鍵の生成（ED25519 推奨）
ssh-keygen -t ed25519 -C "your_email@example.com"

# RSAを使う場合は 4096bit 以上を指定
# ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# 2. SSHエージェントに登録
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. 公開鍵をコピー
cat ~/.ssh/id_ed25519.pub

# 4. GitLab > アバター > Edit profile > Access > SSH keys > Add new key に貼り付け

# 5. 接続テスト
ssh -T git@gitlab.com
# 成功時: "Welcome to GitLab, <username>!"
```

> 参考: [GitLab公式ドキュメント - Use SSH keys with GitLab](https://docs.gitlab.com/ee/user/ssh.html)
