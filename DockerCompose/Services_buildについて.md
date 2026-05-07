# Docker Compose — `services.build` について

> 参考: [公式ドキュメント — Compose Build Specification](https://docs.docker.com/reference/compose-file/build/)

---

## 概要

`build` は、サービスのイメージをソースからビルドする方法を定義するフィールドです。  
`image` との違いは、既存のイメージを使うのではなく **Dockerfile からイメージを作成する** 点です。

`build` は **文字列（簡易記法）** または **オブジェクト（詳細記法）** で指定できます。

---

## 文字列記法（簡易）

```yaml
services:
  app:
    build: ./dir
```

- `./dir` はビルドコンテキストのパスです（Compose ファイルからの相対パス）。
- 指定したディレクトリの **ルートにある `Dockerfile`** が使われます。
- Git リポジトリの URL も指定できます。

```yaml
services:
  webapp:
    build: https://github.com/mycompany/example.git#branch_or_tag:subdirectory
```

---

## オブジェクト記法（詳細）

```yaml
services:
  app:
    build:
      context: ./dir
      dockerfile: app.Dockerfile
      args:
        APP_ENV: production
      target: prod
```

---

## 主要な属性一覧

### `context`

ビルドコンテキストとなるディレクトリのパス、または Git リポジトリの URL を指定します。  
未設定の場合はプロジェクトディレクトリ（`.`）がデフォルトになります。

```yaml
build:
  context: ./dir
```

### `dockerfile`

使用する Dockerfile のパスを指定します。  
パスは `context` からの相対パスで解釈されます。

```yaml
build:
  context: .
  dockerfile: webapp.Dockerfile
```

`dockerfile_inline` と同時指定はできません。

### `dockerfile_inline`

Dockerfile の内容を Compose ファイル内にインラインで記述できます（Docker Compose 2.17.0 以降）。  
`dockerfile` と同時指定はできません。

```yaml
build:
  context: .
  dockerfile_inline: |
    FROM baseimage
    RUN some command
```

### `args`

Dockerfile の `ARG` 命令に渡すビルド引数を定義します。  
マップ形式またはリスト形式で指定できます。

```yaml
# マップ形式
build:
  context: .
  args:
    GIT_COMMIT: cdc3b19
    APP_ENV: production

# リスト形式
build:
  context: .
  args:
    - GIT_COMMIT=cdc3b19
```

値を省略した場合、ビルド時にユーザーが入力するか、環境変数から取得されます。

### `target`

マルチステージビルドで使用するステージ名を指定します。

```yaml
build:
  context: .
  target: prod
```

### `cache_from`

ビルドキャッシュの取得元を指定します。

```yaml
build:
  context: .
  cache_from:
    - alpine:latest
    - type=local,src=path/to/cache
    - type=gha
```

### `cache_to`

ビルドキャッシュのエクスポート先を指定します。

```yaml
build:
  context: .
  cache_to:
    - user/app:cache
    - type=local,dest=path/to/cache
```

### `platforms`

ビルドターゲットのプラットフォームを指定します（マルチプラットフォームビルド）。

```yaml
build:
  context: .
  platforms:
    - linux/amd64
    - linux/arm64
```

---

## `build` と `image` の併用

両方を指定した場合、`pull_policy` の設定に従って動作します。  
`pull_policy` が未設定の場合、Compose はまずレジストリからイメージの取得を試み、見つからなければソースからビルドします。

```yaml
services:
  app:
    image: myapp:latest   # タグ名 / push 先
    build: ./app          # ビルドコンテキスト
```

`image` を省略するとビルドされたイメージに名前が付かないため、`docker compose push` 時に警告が出ます。

---

## まとめ

| 属性                | 説明                                     |
| ------------------- | ---------------------------------------- |
| `context`           | ビルドコンテキストのパスまたは Git URL   |
| `dockerfile`        | 使用する Dockerfile のパス               |
| `dockerfile_inline` | Dockerfile の内容をインラインで記述      |
| `args`              | Dockerfile の `ARG` に渡すビルド引数     |
| `target`            | マルチステージビルドのターゲットステージ |
| `cache_from`        | ビルドキャッシュの取得元                 |
| `cache_to`          | ビルドキャッシュのエクスポート先         |
| `platforms`         | ビルド対象プラットフォーム               |
