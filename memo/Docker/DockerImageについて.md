# Docker Image について

> 参考:
> - [What is an image? — Docker Docs](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
> - [Storage drivers — Docker Docs](https://docs.docker.com/engine/storage/drivers/)
> - [Building best practices — Docker Docs](https://docs.docker.com/build/building/best-practices/)
> - [Multi-stage builds — Docker Docs](https://docs.docker.com/build/building/multi-stage/)
> - [docker image コマンド — Docker Docs](https://docs.docker.com/reference/cli/docker/image/)

---

## Docker Image とは

Docker Image（コンテナイメージ）は、コンテナを実行するために必要な**ファイル・バイナリ・ライブラリ・設定のすべてをまとめた標準化されたパッケージ**です。

イメージには 2 つの重要な性質があります。

| 性質                       | 説明                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| **イミュータブル（不変）** | 一度作成されたイメージは変更できない。変更は新しいイメージとして作成する                   |
| **レイヤー構造**           | イメージは複数のレイヤーの積み重ねで構成される。各レイヤーはファイルシステムへの変更を表す |

コンテナを起動すると、この読み取り専用のイメージの上に薄い書き込み可能なレイヤー（コンテナレイヤー）が追加されます。

```
┌──────────────────────────────┐
│  コンテナレイヤー（書き込み可能）  │  ← コンテナ起動時に追加
├──────────────────────────────┤
│         RUN npm install       │  ← 読み取り専用
├──────────────────────────────┤
│         COPY . /app           │  ← 読み取り専用
├──────────────────────────────┤
│        FROM node:22           │  ← 読み取り専用（ベースイメージ）
└──────────────────────────────┘
```

---

## イメージ名とタグの仕組み

### 命名形式

```
[レジストリホスト/][名前空間/]リポジトリ名[:タグ][@ダイジェスト]
```

| 部分             | 説明                                              | 例                                  |
| ---------------- | ------------------------------------------------- | ----------------------------------- |
| レジストリホスト | イメージの保存先レジストリ（省略時は Docker Hub） | `ghcr.io`, `myregistry.example.com` |
| 名前空間         | Docker Hub ではユーザー名や組織名                 | `myorg`                             |
| リポジトリ名     | イメージの名前                                    | `myapp`                             |
| タグ             | バージョンや用途を表すラベル（省略時は `latest`） | `1.0`, `latest`, `alpine`           |
| ダイジェスト     | イメージの SHA256 ハッシュ（不変の参照）          | `sha256:abc123...`                  |

**例：**

```
ubuntu                          # Docker Hub の公式 ubuntu イメージ（latest タグ）
ubuntu:22.04                    # ubuntu の 22.04 タグ
nginx:1.27-alpine               # nginx の alpine バリアント
myuser/myapp:v2.0               # Docker Hub の個人リポジトリ
ghcr.io/myorg/myapp:main        # GitHub Container Registry
myregistry.example.com/app:1.0  # プライベートレジストリ
```

### タグ と ダイジェスト の違い

|                | タグ                                             | ダイジェスト                               |
| -------------- | ------------------------------------------------ | ------------------------------------------ |
| **変更可能性** | 変更可能（同じタグが別イメージを指すことがある） | 不変（常に同一イメージを指す）             |
| **使用例**     | `ubuntu:22.04`                                   | `ubuntu@sha256:abc123...`                  |
| **用途**       | 日常的な参照・バージョン管理                     | 完全な再現性の保証（セキュリティ用途など） |

### レジストリとリポジトリの違い

| 用語           | 説明                                                                                                       |
| -------------- | ---------------------------------------------------------------------------------------------------------- |
| **レジストリ** | コンテナイメージを保存・配布する集中管理の場所（例：Docker Hub、ECR、ACR）                                 |
| **リポジトリ** | レジストリ内で関連するイメージをまとめたコレクション。1 つのリポジトリに複数のタグ（バージョン）が存在する |

---

## レイヤー構造の仕組み

### Dockerfile とレイヤー

Dockerfile の命令のうち、ファイルシステムを変更するものが新しいレイヤーを生成します。

```dockerfile
# syntax=docker/dockerfile:1
FROM ubuntu:22.04        # ← レイヤー 1（ベースイメージを取得）
LABEL maintainer="me"   # ← レイヤーなし（メタデータのみ）
COPY . /app             # ← レイヤー 2（ファイルの追加）
RUN make /app           # ← レイヤー 3（ビルド実行）
RUN rm -rf /tmp/cache   # ← レイヤー 4（キャッシュ削除）
CMD ["python", "app.py"] # ← レイヤーなし（メタデータのみ）
```

レイヤーを作成する命令 → `RUN`, `COPY`, `ADD`  
レイヤーを作らない命令 → `FROM`, `LABEL`, `ENV`, `CMD`, `ENTRYPOINT`, `EXPOSE`, `WORKDIR`, `USER` など（メタデータのみ）

> 注意: `RUN rm -rf /tmp/cache` で削除しても、前のレイヤーにはファイルが残るためイメージサイズは減りません。  
> 削除は同一 `RUN` 命令内で行うか、マルチステージビルドを使う必要があります。

### レイヤーの共有

複数のイメージが同じレイヤーを共有できます。これにより：

- ディスク使用量が削減される
- `docker pull` 時に既に持っているレイヤーはスキップされる
- ビルドキャッシュが効く

```
イメージ A                  イメージ B
┌────────────────┐         ┌────────────────┐
│  RUN make /app │         │  RUN test.sh   │
├────────────────┤         ├────────────────┤
│ COPY . /app    │         │ COPY . /app    │
├────────────────┤         └───────┬────────┘
│ FROM node:22   │←───共有──────────┘
└────────────────┘
```

### Copy-on-Write（CoW）戦略

コンテナがイメージ内のファイルを変更する場合、Docker はそのファイルをコンテナの書き込みレイヤーにコピーしてから変更します（Copy-on-Write）。

- 変更していないファイルはイメージのレイヤーをそのまま参照する → ディスク使用量を最小化
- 同じイメージから起動した複数のコンテナは、読み取り専用のレイヤーを共有する → 効率的
- 各コンテナは自分専用の書き込みレイヤーを持つ → 独立したデータ状態

```
コンテナ 1    コンテナ 2    コンテナ 3
┌────────┐  ┌────────┐  ┌────────┐
│ 書き込み│  │ 書き込み│  │ 書き込み│   ← 各コンテナが独自に持つ
├────────┴──┴────────┴──┴────────┤
│         読み取り専用レイヤー群       │   ← 共有（ubuntu イメージなど）
└─────────────────────────────────┘
```

---

## `docker image` コマンド一覧

| コマンド               | 説明                                   |
| ---------------------- | -------------------------------------- |
| `docker image ls`      | ローカルのイメージ一覧を表示する       |
| `docker image pull`    | レジストリからイメージを取得する       |
| `docker image push`    | レジストリにイメージをアップロードする |
| `docker image build`   | Dockerfile からイメージをビルドする    |
| `docker image tag`     | イメージに新しいタグを付ける           |
| `docker image rm`      | イメージを削除する                     |
| `docker image prune`   | 使われていないイメージを削除する       |
| `docker image inspect` | イメージの詳細情報を表示する           |
| `docker image history` | イメージのレイヤー履歴を表示する       |
| `docker image save`    | イメージを tar ファイルに保存する      |
| `docker image load`    | tar ファイルからイメージを読み込む     |

### よく使うコマンドの例

```bash
# ローカルのイメージ一覧を表示
docker image ls
docker images  # 省略形

# Docker Hub から nginx の最新版を取得
docker image pull nginx
docker image pull nginx:1.27-alpine

# イメージを削除
docker image rm nginx:1.27-alpine
docker rmi nginx:1.27-alpine  # 省略形

# 使用されていないすべてのイメージを削除
docker image prune
docker image prune -a  # タグのないものも含めてすべて削除

# イメージのレイヤー構造を確認
docker image history myapp:1.0

# イメージの詳細情報を確認（レイヤーのダイジェストなど）
docker image inspect myapp:1.0

# イメージにタグを付ける（別名をつける）
docker image tag myapp:1.0 myuser/myapp:1.0
docker image tag myapp:1.0 myapp:latest

# イメージをファイルに保存して別環境に持ち込む
docker image save myapp:1.0 -o myapp.tar
docker image load -i myapp.tar
```

---

## イメージのビルド（Dockerfile）

### 基本的な Dockerfile の構造

```dockerfile
# syntax=docker/dockerfile:1

# ベースイメージを指定
FROM node:22-alpine

# 作業ディレクトリを設定
WORKDIR /app

# 依存関係ファイルをコピーしてインストール（キャッシュ効率を高めるため先に行う）
COPY package*.json ./
RUN npm ci --only=production

# アプリケーションのソースコードをコピー
COPY . .

# ポートを宣言（ドキュメント目的。実際の公開は docker run -p で行う）
EXPOSE 3000

# コンテナ起動時のコマンド
CMD ["node", "server.js"]
```

### 主要な Dockerfile 命令

| 命令         | 説明                                                                       |
| ------------ | -------------------------------------------------------------------------- |
| `FROM`       | ベースイメージを指定する。すべての Dockerfile の最初に書く                 |
| `WORKDIR`    | 以降の命令の作業ディレクトリを設定する                                     |
| `COPY`       | ホストからコンテナにファイルをコピーする                                   |
| `ADD`        | COPY と同様だが、URL からの取得や tar の自動展開もできる                   |
| `RUN`        | ビルド時にコマンドを実行し、その結果を新しいレイヤーとして保存する         |
| `CMD`        | コンテナ起動時のデフォルトコマンドを指定する（`docker run` で上書き可）    |
| `ENTRYPOINT` | コンテナのメインコマンドを固定する（`CMD` との組み合わせで使うことが多い） |
| `ENV`        | 環境変数を設定する                                                         |
| `ARG`        | ビルド時のみ有効な変数を定義する（`--build-arg` で渡す）                   |
| `EXPOSE`     | コンテナがリッスンするポートを宣言する（ドキュメント用途）                 |
| `VOLUME`     | 永続化すべきマウントポイントを宣言する                                     |
| `USER`       | 以降の命令を実行するユーザーを変更する                                     |
| `LABEL`      | イメージにメタデータ（ラベル）を付ける                                     |

### `docker build` の基本

```bash
# カレントディレクトリの Dockerfile でビルド
docker build -t myapp:1.0 .

# 別のファイル名を指定
docker build -f Dockerfile.prod -t myapp:prod .

# ビルド時変数を渡す
docker build --build-arg NODE_ENV=production -t myapp:prod .

# キャッシュを使わず再ビルド
docker build --no-cache -t myapp:latest .

# 最新のベースイメージを取得してビルド
docker build --pull -t myapp:latest .
```

---

## マルチステージビルド

ビルド用の環境と実行用の環境を分離することで、最終イメージを軽量に保つ手法です。

### 使用例（Go アプリケーション）

```dockerfile
# syntax=docker/dockerfile:1

# ステージ 1: ビルド
FROM golang:1.25 AS build
WORKDIR /src
COPY . .
RUN go build -o /bin/app ./cmd/app

# ステージ 2: 実行（軽量イメージ）
FROM scratch
COPY --from=build /bin/app /bin/app
CMD ["/bin/app"]
```

ステージ 1（`golang:1.25`）の Go SDK やビルドツールは最終イメージに含まれません。

### 使用例（Node.js アプリケーション）

```dockerfile
# syntax=docker/dockerfile:1

# ステージ 1: 依存関係インストール
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# ステージ 2: ビルド
FROM deps AS builder
COPY . .
RUN npm run build

# ステージ 3: 本番用（最小限のファイルのみ）
FROM node:22-alpine AS runner
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

### 特定ステージだけビルドする

```bash
# "build" ステージまでビルド（デバッグや中間確認に使う）
docker build --target build -t myapp:build-only .
```

---

## Docker Hub と主要なレジストリ

### Docker Hub の信頼コンテンツ

| 種別                             | 説明                                                                                                                              |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Docker Official Images**       | Docker が厳選・管理する公式イメージ（`ubuntu`, `nginx`, `postgres` など）。セキュリティの基準が高く、多くのユーザーの出発点となる |
| **Docker Hardened Images**       | CVE がほぼゼロの最小限・本番向けイメージ。Apache 2.0 で提供                                                                       |
| **Verified Publisher**           | Docker が認証した商用パブリッシャーの高品質イメージ                                                                               |
| **Docker-Sponsored Open Source** | Docker がスポンサーするオープンソースプロジェクトのイメージ                                                                       |

### 主要なレジストリ

| レジストリ                                | 説明                             |
| ----------------------------------------- | -------------------------------- |
| **Docker Hub** (`docker.io`)              | デフォルトのパブリックレジストリ |
| **GitHub Container Registry** (`ghcr.io`) | GitHub に統合されたレジストリ    |
| **Amazon ECR**                            | AWS のプライベートレジストリ     |
| **Azure Container Registry** (ACR)        | Azure のプライベートレジストリ   |
| **Google Artifact Registry** (GAR)        | GCP のレジストリ                 |

### pull / push の手順

```bash
# Docker Hub にログイン
docker login

# イメージをプッシュ（事前に docker build でイメージを作成しておく）
docker image tag myapp:1.0 myuser/myapp:1.0
docker image push myuser/myapp:1.0

# 別のレジストリにプッシュ
docker image tag myapp:1.0 ghcr.io/myorg/myapp:1.0
docker image push ghcr.io/myorg/myapp:1.0
```

---

## ベストプラクティス

### 1. 適切なベースイメージを選ぶ

- できるだけ小さく最小限のイメージを選ぶ（`alpine` バリアントなど）
- Docker Official Images や Verified Publisher のイメージを使う
- ビルド用と本番用で異なるイメージを使う（マルチステージビルド）

```dockerfile
# 悪い例: フル ubuntu イメージを使っている
FROM ubuntu:22.04
RUN apt-get install -y python3

# 良い例: 公式 python イメージを使う
FROM python:3.12-slim
```

### 2. `.dockerignore` を活用する

不要なファイルをビルドコンテキストから除外することで、ビルドが速くなりイメージが小さくなります。

```
# .dockerignore の例
node_modules/
.git/
*.md
.env
dist/
__tests__/
```

### 3. レイヤーキャッシュを活用する

変更が少ないものを先にコピーし、変更が多いものを後にコピーすることでキャッシュが効きやすくなります。

```dockerfile
# 良い例: 依存関係（変更頻度が低い）を先にコピー
COPY package*.json ./
RUN npm ci

# ソースコード（変更頻度が高い）を後にコピー
COPY . .
```

```dockerfile
# 悪い例: ソースコードが変わるたびに npm install も再実行される
COPY . .
RUN npm ci
```

### 4. `RUN` 命令はまとめる

複数の `RUN` を `&&` でつなぐことで中間レイヤーを削減し、イメージを小さく保てます。

```dockerfile
# 良い例: 1つの RUN でインストール・クリーンアップを行う
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

# 悪い例: クリーンアップしても前のレイヤーにキャッシュが残る
RUN apt-get update
RUN apt-get install -y curl git
RUN rm -rf /var/lib/apt/lists/*
```

### 5. 不要なパッケージをインストールしない

`--no-install-recommends`（apt）など、余分な依存関係を省くオプションを使う。

### 6. root 以外のユーザーで実行する（セキュリティ）

```dockerfile
RUN groupadd -r appuser && useradd --no-log-init -r -g appuser appuser
USER appuser
```

### 7. ベースイメージのバージョンをピン留めする

本番環境では予期せぬ変更を防ぐため、タグではなくダイジェストで固定することも検討する。

```dockerfile
# タグ指定（publisher が更新すると変わる可能性がある）
FROM alpine:3.21

# ダイジェスト指定（完全に固定）
FROM alpine:3.21@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
```

---

## イメージ・コンテナ・ボリュームの関係まとめ

```
┌──────────────────────────────────────────────────────┐
│                   Docker Image                        │
│  （読み取り専用レイヤーの積み重ね。イミュータブル）         │
└────────────────────┬─────────────────────────────────┘
                     │ docker run で起動
                     ▼
┌──────────────────────────────────────────────────────┐
│                  Container                            │
│  ┌────────────────────────────────────────────────┐  │
│  │  書き込みレイヤー（コンテナ削除時に消える）         │  │
│  └────────────────────────────────────────────────┘  │
│  ＋ イメージの読み取り専用レイヤー（共有）              │
└──────────────────────────────────────────────────────┘
         ↕ マウント
┌──────────────────────────────────────────────────────┐
│             Volume / Bind Mount                       │
│  （コンテナ外に永続化。コンテナ削除後も残る）             │
└──────────────────────────────────────────────────────┘
```

|                    | Image              | Container                  | Volume                 |
| ------------------ | ------------------ | -------------------------- | ---------------------- |
| **データの永続性** | 永続（変更不可）   | 停止後も残る、削除で消える | コンテナと独立して永続 |
| **変更可能性**     | 不変               | 書き込みレイヤーで変更可   | 読み書き可能           |
| **共有**           | 複数コンテナで共有 | 個別                       | 複数コンテナで共有可   |
