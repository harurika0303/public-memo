# GitLab SSH接続の正しい設定手順

> 参考: [GitLab公式ドキュメント - Use SSH keys with GitLab](https://docs.gitlab.com/ee/user/ssh.html)

---

## 概要

SSH接続を使うことで、GitLabへのpush/pull時にユーザー名・パスワードを入力せずに安全に通信できる。

手順は以下の3ステップ。

1. ローカルにSSH鍵ペアを生成する
2. 公開鍵をGitLabアカウントに登録する
3. 接続を確認する

---

## 前提条件

- OpenSSHクライアントがインストールされていること（Ubuntu/WSLには標準搭載）
- SSHのバージョンが 6.5 以上であること

```bash
ssh -V
```

---

## Step 1. 既存のSSH鍵を確認する

新しく鍵を生成する前に、既存の鍵があるか確認する。

```bash
ls ~/.ssh/
```

以下のいずれかのファイルが存在すれば、すでに鍵がある。

| キータイプ         | 公開鍵           | 秘密鍵       |
| ------------------ | ---------------- | ------------ |
| ED25519（推奨）    | `id_ed25519.pub` | `id_ed25519` |
| RSA（4096bit以上） | `id_rsa.pub`     | `id_rsa`     |

既存の鍵を使う場合は Step 2 をスキップして Step 3 へ。

---

## Step 2. SSH鍵ペアを生成する

**ED25519を推奨**（RSAより安全・高速）。

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

RSAを使う場合は 4096bit 以上を指定する。

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

コマンド実行後は以下のプロンプトに従う。

```
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
# そのままEnterでデフォルトパスに保存

Enter passphrase (empty for no passphrase):
# パスフレーズを設定（推奨）またはEnterでスキップ
```

---

## Step 3. SSHエージェントに秘密鍵を登録する

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
# RSAの場合: ssh-add ~/.ssh/id_rsa
```

---

## Step 4. 公開鍵をGitLabに登録する

公開鍵の内容を表示してコピーする。

```bash
cat ~/.ssh/id_ed25519.pub
# RSAの場合: cat ~/.ssh/id_rsa.pub
```

GitLabでの登録手順：

1. 右上のアバターをクリック
2. **Edit profile** を選択
3. 左サイドバーの **Access > SSH keys** を選択
4. **Add new key** をクリック
5. **Key** ボックスにコピーした公開鍵を貼り付ける
6. **Title** に識別名を入力（例: `Work WSL`）
7. **Add key** をクリック

---

## Step 5. 接続を確認する

```bash
ssh -T git@gitlab.com
```

初回接続時はホストの認証を確認するプロンプトが表示される場合がある。

```
The authenticity of host 'gitlab.com' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

`yes` と入力すると、以下のメッセージが表示されれば成功。

```
Welcome to GitLab, <username>!
```

---

## トラブルシューティング

接続できない場合は `-v` オプションで詳細ログを確認する。

```bash
ssh -vT git@gitlab.com
```

よくある原因：
- `~/.ssh/` のパーミッション不足 → 以下のコマンドで修正
  ```bash
  chmod 700 ~/.ssh
  chmod 600 ~/.ssh/id_ed25519      # 秘密鍵
  chmod 644 ~/.ssh/id_ed25519.pub  # 公開鍵
  # RSAの場合: chmod 600 ~/.ssh/id_rsa && chmod 644 ~/.ssh/id_rsa.pub
  ```
- GitLabに登録した公開鍵と手元の秘密鍵が不一致（WSL再インストール後に多い）
- SSHエージェントに鍵が登録されていない

---

## 参考

- [Use SSH keys with GitLab - GitLab公式ドキュメント](https://docs.gitlab.com/ee/user/ssh.html)
- [Advanced SSH key configuration - GitLab公式ドキュメント](https://docs.gitlab.com/user/ssh_advanced/)
- [SSH troubleshooting - GitLab公式ドキュメント](https://docs.gitlab.com/user/ssh_troubleshooting/)
