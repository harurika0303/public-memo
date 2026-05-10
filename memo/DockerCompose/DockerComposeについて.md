# Docker Compose について

> 参考: [公式ドキュメント — Compose file reference](https://docs.docker.com/reference/compose-file/)  
> 参考: [公式ドキュメント — Docker Compose overview](https://docs.docker.com/compose/)

## Docker Compose とは

Docker Compose は、複数のコンテナを定義・実行するためのツールです。  
`compose.yaml` という YAML ファイルにサービス構成を記述し、1 つのコマンドでアプリケーション全体を起動・停止できます。

例えば「Web サーバー + アプリサーバー + DB」という構成を一括で管理する場合に使います。

---

## 基本的な構成要素

### services

アプリを構成する各コンテナを定義するセクションです。

```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
  app:
    build: ./app
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
```

| 項目          | 説明                                                       |
| ------------- | ---------------------------------------------------------- |
| `image`       | 使用する Docker イメージを指定する                         |
| `build`       | Dockerfile からイメージをビルドするパスを指定する          |
| `ports`       | `ホスト側ポート:コンテナ側ポート` の形式でポートを公開する |
| `environment` | 環境変数を設定する                                         |
| `depends_on`  | 起動順序の依存関係を指定する                               |
| `volumes`     | ボリュームをマウントする                                   |

### volumes

データを永続化したり、コンテナ間でファイルを共有するために使います。

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

### networks

コンテナ間の通信ネットワークを定義します。  
デフォルトでは同一 Compose ファイル内のサービスは同じネットワークに属するため、サービス名で名前解決できます。

```yaml
networks:
  backend:
    driver: bridge
```

---

## よく使うコマンド

| コマンド                              | 説明                                               |
| ------------------------------------- | -------------------------------------------------- |
| `docker compose up`                   | サービスを起動する（`-d` でバックグラウンド実行）  |
| `docker compose down`                 | サービスを停止してコンテナ・ネットワークを削除する |
| `docker compose build`                | イメージをビルドする                               |
| `docker compose ps`                   | 起動中のサービス一覧を表示する                     |
| `docker compose logs`                 | ログを表示する（`-f` でリアルタイム追従）          |
| `docker compose exec <service> <cmd>` | 起動中のコンテナでコマンドを実行する               |
| `docker compose restart`              | サービスを再起動する                               |

---

## ファイル名について

| ファイル名            | 説明                                                  |
| --------------------- | ----------------------------------------------------- |
| `compose.yaml`        | **推奨**されるファイル名（公式仕様）                  |
| `compose.yml`         | 利用可能だが `compose.yaml` が優先される              |
| `docker-compose.yaml` | 利用可能だが非推奨                                    |
| `docker-compose.yml`  | 利用可能だが非推奨（旧 Docker Compose v1 時代の命名） |

複数環境の設定を分割する場合は `compose.override.yaml` や `-f` オプションで複数ファイルを指定できます。

```bash
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

---

## depends_on の注意点

`depends_on` はコンテナの**起動順序**を制御しますが、サービスが「準備完了」になるまで待つわけではありません。  
DB の起動完了を待ってからアプリを起動したい場合は `healthcheck` と組み合わせます。

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

---

## まとめ

| 概念         | 役割                 |
| ------------ | -------------------- |
| `services`   | 各コンテナの定義     |
| `volumes`    | データの永続化・共有 |
| `networks`   | コンテナ間通信の設定 |
| `depends_on` | 起動順序の制御       |

Docker Compose を使うことで、複雑なマルチコンテナ構成を 1 つのファイルで管理でき、開発環境の再現性や共有が容易になります。
