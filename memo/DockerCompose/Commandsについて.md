# Docker Compose — よく使うコマンド

> 参考: [公式ドキュメント — docker compose](https://docs.docker.com/reference/cli/docker/compose/)

---

## コマンド一覧（早見表）

| コマンド                 | 説明                                        |
| ------------------------ | ------------------------------------------- |
| `docker compose up`      | コンテナを作成・起動する                    |
| `docker compose down`    | コンテナを停止・削除する                    |
| `docker compose build`   | イメージをビルドする                        |
| `docker compose ps`      | 起動中のコンテナ一覧を表示する              |
| `docker compose logs`    | ログを表示する                              |
| `docker compose exec`    | 起動中のコンテナでコマンドを実行する        |
| `docker compose run`     | サービスに対して1回限りのコマンドを実行する |
| `docker compose restart` | サービスを再起動する                        |
| `docker compose stop`    | コンテナを停止する（削除はしない）          |
| `docker compose start`   | 停止中のコンテナを起動する                  |
| `docker compose pull`    | サービスのイメージを取得する                |
| `docker compose config`  | Compose ファイルを解析・検証して表示する    |

---

## `docker compose up`

> 参考: [docker compose up](https://docs.docker.com/reference/cli/docker/compose/up/)

```
docker compose up [OPTIONS] [SERVICE...]
```

コンテナのビルド・作成・起動とログの表示を一括で行います。  
サービス名を指定すると、そのサービスだけを起動できます。

コマンド終了時（Ctrl+C など）にすべてのコンテナが停止します。

### よく使うオプション

| オプション                   | 説明                                                               |
| ---------------------------- | ------------------------------------------------------------------ |
| `-d`, `--detach`             | バックグラウンドで起動する（ログを表示しない）                     |
| `--build`                    | 起動前にイメージをビルドする                                       |
| `--no-build`                 | イメージをビルドせず、既存のものを使う                             |
| `--force-recreate`           | 設定・イメージが変わっていなくてもコンテナを強制的に再作成する     |
| `--no-recreate`              | すでにコンテナが存在する場合は再作成しない                         |
| `--remove-orphans`           | Compose ファイルに定義されていないサービスのコンテナを削除する     |
| `--wait`                     | すべてのサービスが起動/healthy になるまで待つ（`-d` を暗黙に含む） |
| `-V`, `--renew-anon-volumes` | 以前のコンテナの匿名ボリュームを引き継がず、新規作成する           |
| `--scale SERVICE=NUM`        | サービスのコンテナ数を指定して起動する                             |

### 使用例

```bash
# バックグラウンドで全サービスを起動
docker compose up -d

# イメージを再ビルドしてから起動
docker compose up --build

# app サービスだけ起動
docker compose up app

# すべてのサービスが healthy になるまで待つ
docker compose up --wait
```

---

## `docker compose down`

> 参考: [docker compose down](https://docs.docker.com/reference/cli/docker/compose/down/)

```
docker compose down [OPTIONS] [SERVICES]
```

コンテナを停止し、削除します。ネットワークも削除されます。  
**ボリュームはデフォルトでは削除されません**（データを保持するため）。

デフォルトで削除されるもの：
- サービスのコンテナ
- `networks` セクションで定義したネットワーク
- デフォルトネットワーク

削除されないもの：
- `external: true` のネットワーク・ボリューム
- 匿名ボリューム（デフォルト）

### よく使うオプション

| オプション         | 説明                                                                       |
| ------------------ | -------------------------------------------------------------------------- |
| `-v`, `--volumes`  | `volumes` セクションで定義した名前付きボリュームと匿名ボリュームも削除する |
| `--rmi local`      | カスタムタグのない（ビルドした）イメージを削除する                         |
| `--rmi all`        | サービスが使うすべてのイメージを削除する                                   |
| `--remove-orphans` | Compose ファイルに定義されていないサービスのコンテナも削除する             |
| `-t`, `--timeout`  | コンテナ停止のタイムアウト秒数を指定する（デフォルト: 10秒）               |

### 使用例

```bash
# コンテナとネットワークを削除（ボリュームは残す）
docker compose down

# コンテナ・ネットワーク・ボリュームをすべて削除
docker compose down -v

# ビルドしたイメージも一緒に削除
docker compose down --rmi local

# 完全クリーンアップ（ボリューム・イメージ・孤立コンテナも削除）
docker compose down -v --rmi all --remove-orphans
```

---

## `docker compose build`

> 参考: [docker compose build](https://docs.docker.com/reference/cli/docker/compose/build/)

```
docker compose build [OPTIONS] [SERVICE...]
```

`build` セクションを持つサービスのイメージをビルド（または再ビルド）します。  
デフォルトで `project-service` という名前でタグ付けされます。

Dockerfile や build コンテキストの内容を変更した場合は `docker compose build` を実行して再ビルドしてください。

### よく使うオプション

| オプション            | 説明                                         |
| --------------------- | -------------------------------------------- |
| `--no-cache`          | キャッシュを使わずにビルドする               |
| `--pull`              | 常に最新のベースイメージを取得してビルドする |
| `--build-arg KEY=VAL` | ビルド時変数を渡す                           |
| `-q`, `--quiet`       | ビルド出力を抑制する                         |

### 使用例

```bash
# すべてのサービスをビルド
docker compose build

# キャッシュなしで再ビルド
docker compose build --no-cache

# app サービスだけビルド
docker compose build app
```

---

## `docker compose ps`

> 参考: [docker compose ps](https://docs.docker.com/reference/cli/docker/compose/ps/)

```
docker compose ps [OPTIONS] [SERVICE...]
```

プロジェクトに属するコンテナの一覧と状態を表示します。

### よく使うオプション

| オプション    | 説明                                                          |
| ------------- | ------------------------------------------------------------- |
| `-a`, `--all` | 停止中のコンテナも含めて表示する                              |
| `--status`    | 指定した状態のコンテナだけ表示する（例: `running`, `exited`） |
| `--format`    | 出力フォーマットを指定する（例: `json`, `table`）             |

### 使用例

```bash
# 起動中のコンテナを表示
docker compose ps

# 停止中も含めて表示
docker compose ps -a
```

---

## `docker compose logs`

> 参考: [docker compose logs](https://docs.docker.com/reference/cli/docker/compose/logs/)

```
docker compose logs [OPTIONS] [SERVICE...]
```

サービスのログを表示します。サービス名を省略するとすべてのサービスのログを表示します。

### よく使うオプション

| オプション           | 説明                                                                  |
| -------------------- | --------------------------------------------------------------------- |
| `-f`, `--follow`     | リアルタイムでログを追従する（`tail -f` 相当）                        |
| `-n`, `--tail N`     | 末尾から N 行だけ表示する（デフォルト: すべて）                       |
| `-t`, `--timestamps` | タイムスタンプを表示する                                              |
| `--since DURATION`   | 指定した時刻以降のログを表示する（例: `30m`, `2024-01-01T00:00:00Z`） |
| `--until DURATION`   | 指定した時刻以前のログを表示する                                      |

### 使用例

```bash
# すべてのサービスのログをリアルタイムで表示
docker compose logs -f

# app サービスの末尾 100 行を表示
docker compose logs --tail 100 app

# タイムスタンプ付きで表示
docker compose logs -t

# 直近 30 分のログを表示
docker compose logs --since 30m
```

---

## `docker compose exec`

> 参考: [docker compose exec](https://docs.docker.com/reference/cli/docker/compose/exec/)

```
docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

**起動中のコンテナ**に対してコマンドを実行します。デフォルトで TTY を確保するため、インタラクティブなシェルも使えます。

`docker exec` の Compose 版です。`docker exec` と異なり、デフォルトでインタラクティブ・TTY が有効です。

### よく使うオプション

| オプション             | 説明                                           |
| ---------------------- | ---------------------------------------------- |
| `-u`, `--user USER`    | 指定したユーザーでコマンドを実行する           |
| `-w`, `--workdir PATH` | 実行時のワーキングディレクトリを指定する       |
| `-e`, `--env KEY=VAL`  | 環境変数を設定する                             |
| `-d`, `--detach`       | バックグラウンドでコマンドを実行する           |
| `-T`, `--no-tty`       | TTY を無効にする（スクリプト内で使う場合など） |

### 使用例

```bash
# app コンテナで bash を起動
docker compose exec app bash

# db コンテナで psql を実行
docker compose exec db psql -U postgres

# root ユーザーでコマンドを実行
docker compose exec -u root app sh
```

---

## `docker compose run`

> 参考: [docker compose run](https://docs.docker.com/reference/cli/docker/compose/run/)

```
docker compose run [OPTIONS] SERVICE [COMMAND] [ARGS...]
```

**新しいコンテナを作成して**1回限りのコマンドを実行します。`exec` とは異なり、実行中のコンテナは必要ありません。

`exec` との違い：
- `exec`：**起動中のコンテナ**に対してコマンドを実行する
- `run`：**新しいコンテナを作成して**コマンドを実行する

デフォルトでは `ports` の公開は行いません（既存のポートと衝突を避けるため）。

### よく使うオプション

| オプション              | 説明                               |
| ----------------------- | ---------------------------------- |
| `--rm`                  | 実行後にコンテナを自動削除する     |
| `-d`, `--detach`        | バックグラウンドで実行する         |
| `--no-deps`             | 依存サービスを起動しない           |
| `-e`, `--env KEY=VAL`   | 環境変数を設定する                 |
| `--entrypoint CMD`      | エントリーポイントを上書きする     |
| `-P`, `--service-ports` | サービスのポートをホストに公開する |
| `-u`, `--user USER`     | 指定したユーザーで実行する         |
| `-w`, `--workdir PATH`  | ワーキングディレクトリを指定する   |

### 使用例

```bash
# マイグレーションを実行して終了後にコンテナを削除
docker compose run --rm app python manage.py migrate

# シェルを起動（依存サービスなし）
docker compose run --rm --no-deps app bash

# db コンテナで psql を実行
docker compose run --rm db psql -h db -U postgres
```

---

## `docker compose restart`

> 参考: [docker compose restart](https://docs.docker.com/reference/cli/docker/compose/restart/)

```
docker compose restart [OPTIONS] [SERVICE...]
```

サービスのコンテナを再起動します。設定変更は**反映されません**（設定変更を反映するには `up` を使ってください）。

| オプション          | 説明                                           |
| ------------------- | ---------------------------------------------- |
| `-t`, `--timeout N` | タイムアウト秒数を指定する（デフォルト: 10秒） |

---

## `docker compose stop` / `start`

| コマンド                            | 説明                                                                 |
| ----------------------------------- | -------------------------------------------------------------------- |
| `docker compose stop [SERVICE...]`  | コンテナを停止する（コンテナ・ネットワーク・ボリュームは削除しない） |
| `docker compose start [SERVICE...]` | 停止中のコンテナを起動する                                           |

`down` との違い：`stop` はコンテナを停止するだけで削除しません。`start` で再起動できます。

```bash
# db だけ停止
docker compose stop db

# db だけ再開
docker compose start db
```

---

## `docker compose pull`

```
docker compose pull [OPTIONS] [SERVICE...]
```

サービスで使用するイメージをレジストリから取得します。

```bash
# すべてのサービスのイメージを取得
docker compose pull

# db のイメージだけ取得
docker compose pull db
```

---

## `docker compose config`

```
docker compose config [OPTIONS]
```

Compose ファイルを解析・変数展開して正規化した形式で出力します。設定が正しいか確認するのに使います。

```bash
# Compose ファイルの内容を確認（変数展開済み）
docker compose config

# 構文エラーのチェックのみ
docker compose config --quiet
```

---

## グローバルオプション

コマンドの前に指定するオプションです。

| オプション                  | 説明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `-f`, `--file FILE`         | 使用する Compose ファイルを指定する（複数指定可）            |
| `-p`, `--project-name NAME` | プロジェクト名を指定する                                     |
| `--env-file FILE`           | 使用する環境変数ファイルを指定する                           |
| `--profile PROFILE`         | 有効にするプロファイルを指定する                             |
| `--dry-run`                 | ドライランモード（実際には変更せずコマンドの動作を確認する） |

```bash
# 別のファイルを指定して起動
docker compose -f compose.prod.yaml up -d

# プロジェクト名を指定
docker compose -p myproject up -d

# 複数ファイルをマージして起動
docker compose -f compose.yaml -f compose.override.yaml up -d
```
