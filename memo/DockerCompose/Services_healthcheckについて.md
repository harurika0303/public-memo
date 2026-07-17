# Docker Compose — `services.healthcheck` について

> 参考: [公式ドキュメント — Define services in Docker Compose # healthcheck](https://docs.docker.com/reference/compose-file/services/#healthcheck)  
> 参考: [Dockerfile リファレンス — HEALTHCHECK](https://docs.docker.com/reference/dockerfile/#healthcheck)

---

## 概要

`healthcheck` は、コンテナが「healthy（正常）」かどうかを判定するためのチェックを定義します。  
Dockerfile の `HEALTHCHECK` 命令と同じ仕組みで動作し、Compose ファイルで上書きできます。

`depends_on` の `condition: service_healthy` と組み合わせることで、依存サービスの準備完了を待ってから起動する構成が可能になります。

---

## 基本的な記述例

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
  start_interval: 5s
```

---

## 各フィールドの説明

### `test`

ヘルスチェックで実行するコマンドを指定します。  
**文字列** または **リスト** で指定できます。

#### リスト形式

リストの最初の要素は `NONE`、`CMD`、`CMD-SHELL` のいずれかでなければなりません。

```yaml
# CMD: 引数をそのまま exec で実行する
test: ["CMD", "curl", "-f", "http://localhost"]

# CMD-SHELL: コンテナのデフォルトシェル（Linux では /bin/sh）経由で実行する
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]

# NONE: ヘルスチェックを無効化する
test: ["NONE"]
```

#### 文字列形式

文字列で指定した場合、`CMD-SHELL` を指定したのと同等になります。

```yaml
test: "curl -f http://localhost || exit 1"
```

---

### `interval`

チェックの実行間隔を指定します。  
デフォルト値はイメージの設定に依存します。

```yaml
interval: 30s     # 30 秒ごとにチェック
interval: 1m30s   # 1 分 30 秒ごとにチェック
```

---

### `timeout`

1 回のチェックがこの時間内に完了しなければ失敗とみなします。  
デフォルト値はイメージの設定に依存します。

```yaml
timeout: 10s
```

---

### `retries`

チェックが連続して何回失敗したら `unhealthy` とみなすかを指定します。  
デフォルト値はイメージの設定に依存します。

```yaml
retries: 3
```

---

### `start_period`

コンテナ起動直後のウォームアップ期間を指定します。  
この期間中にチェックが失敗しても `retries` のカウントに含まれません。  
ただし、この期間中にチェックが成功した場合はただちに `healthy` になります。

```yaml
start_period: 40s
```

---

### `start_interval`

`start_period` 中のチェック間隔を指定します（Docker Compose 2.20.2 以降）。  
通常の `interval` より短く設定することで、起動直後の確認を頻繁に行えます。

```yaml
start_interval: 5s
```

---

## 時間の書き方（Duration 形式）

`interval`、`timeout`、`start_period`、`start_interval` はすべて Duration 形式で指定します。

| 単位 | 例                    |
| ---- | --------------------- |
| `us` | `500us`（マイクロ秒） |
| `ms` | `200ms`（ミリ秒）     |
| `s`  | `10s`（秒）           |
| `m`  | `5m`（分）            |
| `h`  | `1h`（時間）          |

組み合わせも可能です: `1m30s`、`2h30m`

---

## ヘルスチェックの無効化

イメージに組み込まれたヘルスチェックを無効にするには、`disable: true` を使います。

```yaml
healthcheck:
  disable: true
```

または `test: ["NONE"]` でも同様に無効化できます。

> `disable: true` を使う場合、他のフィールド（`test` など）は同時に指定できません。

---

## よく使うパターン集

### HTTP エンドポイントの確認

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 10s
```

### PostgreSQL の接続確認

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 5s
  timeout: 5s
  retries: 5
  start_period: 10s
```

### MySQL の接続確認

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
  interval: 10s
  timeout: 5s
  retries: 5
```

### Redis の接続確認

```yaml
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  timeout: 3s
  retries: 3
```

---

## `depends_on` との組み合わせ

`condition: service_healthy` を指定することで、`healthcheck` が `healthy` になるまで依存サービスの起動を待てます。

```yaml
services:
  app:
    image: myapp
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:18
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s
```

> **注意**: `condition: service_healthy` を使う場合、依存先に `healthcheck` が設定されていないとエラーになります。

---

## まとめ

| フィールド       | 説明                                           | デフォルト     |
| ---------------- | ---------------------------------------------- | -------------- |
| `test`           | チェックコマンド（必須）                       | イメージの設定 |
| `interval`       | チェック間隔                                   | イメージの設定 |
| `timeout`        | タイムアウト時間                               | イメージの設定 |
| `retries`        | 失敗回数の閾値                                 | イメージの設定 |
| `start_period`   | 起動ウォームアップ期間                         | イメージの設定 |
| `start_interval` | ウォームアップ中のチェック間隔（v2.20.2 以降） | イメージの設定 |
| `disable`        | `true` でヘルスチェックを無効化                | `false`        |
