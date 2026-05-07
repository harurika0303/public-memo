# Docker Compose — `services.depends_on` について

> 参考: [公式ドキュメント — Define services in Docker Compose # depends_on](https://docs.docker.com/reference/compose-file/services/#depends_on)

---

## 概要

`depends_on` は、サービス間の **起動順序と停止順序** を制御するためのフィールドです。  
依存関係を定義することで、あるサービスが別のサービスより先に起動・後に停止するようになります。

---

## 短縮記法（Short syntax）

依存するサービス名をリストで指定します。

```yaml
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres:18
```

この場合の動作：
- **起動時**: `db` と `redis` が先に起動され、その後 `web` が起動します。
- **停止時**: `web` が先に停止され、その後 `db` と `redis` が停止します。

> **重要**: 短縮記法では、Compose は依存サービスが **「起動した（started）」** ことを確認するだけです。  
> サービスが「準備完了（healthy）」になるまで待ちません。

---

## 詳細記法（Long syntax）

短縮記法では表現できない詳細な条件を設定できます。

```yaml
services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
        restart: true
      redis:
        condition: service_started
  redis:
    image: redis
  db:
    image: postgres:18
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

### `condition` — 依存条件

| 値                               | 説明                                                                        |
| -------------------------------- | --------------------------------------------------------------------------- |
| `service_started`                | 依存サービスのコンテナが起動していれば OK（短縮記法と同等）                 |
| `service_healthy`                | 依存サービスの `healthcheck` が healthy になるまで待つ                      |
| `service_completed_successfully` | 依存サービスが正常終了（exit code 0）するまで待つ。初期化コンテナなどに使用 |

### `restart`

`true` にすると、依存サービスが更新・再起動されたとき、このサービスも再起動します（Docker Compose 2.17.0 以降）。

```yaml
depends_on:
  db:
    condition: service_healthy
    restart: true
```

### `required`

`false` にすると、依存サービスが起動していない・利用できない場合でも警告のみで続行します（Docker Compose 2.20.0 以降）。  
デフォルトは `true`（依存サービスがなければエラー）。

```yaml
depends_on:
  db:
    condition: service_healthy
    required: false
```

---

## `service_healthy` を使う場合の注意点

`condition: service_healthy` を指定する場合、依存先のサービスに必ず **`healthcheck`** を設定してください。  
`healthcheck` がないと Compose はエラーを返します。

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:18
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
```

---

## まとめ

| ポイント                         | 内容                                                     |
| -------------------------------- | -------------------------------------------------------- |
| 起動順序の制御                   | 指定したサービスが先に起動される                         |
| 停止順序の制御                   | 依存するサービスが先に停止される                         |
| 短縮記法                         | 起動したことのみを確認（準備完了は待たない）             |
| `service_healthy`                | `healthcheck` が必須。DB などの準備完了を待つ場合に使う  |
| `service_completed_successfully` | 初期化処理など、正常終了を待つ場合に使う                 |
| `required: false`                | 依存サービスがなくても続行（Docker Compose 2.20.0 以降） |
