# Docker Compose — `services.volumes` について

> 参考: [公式ドキュメント — Define services in Docker Compose # volumes](https://docs.docker.com/reference/compose-file/services/#volumes)

---

## 概要

`volumes` は、ホストパスまたは名前付きボリュームをコンテナにマウントするためのフィールドです。  
データの永続化、コンテナ間のファイル共有、ホストとのファイル同期などに使用します。

マウントの種類には `volume`、`bind`、`tmpfs`、`image`、`npipe` があります。

---

## 短縮記法（Short syntax）

コロン区切りの文字列で指定します。

```
VOLUME:CONTAINER_PATH[:ACCESS_MODE]
```

- `VOLUME`: ホスト側のパス（バインドマウント）または名前付きボリュームの名前
- `CONTAINER_PATH`: コンテナ内のマウント先パス
- `ACCESS_MODE`: アクセス権（省略時は `rw`）

```yaml
services:
  app:
    volumes:
      - /host/data:/container/data          # バインドマウント（絶対パス）
      - ./local-dir:/container/dir          # バインドマウント（相対パス）
      - myvolume:/var/lib/data              # 名前付きボリューム
      - myvolume:/var/lib/data:ro           # 読み取り専用
```

### ACCESS_MODE の選択肢

| 値   | 説明                                                                        |
| ---- | --------------------------------------------------------------------------- |
| `rw` | 読み書き可能（デフォルト）                                                  |
| `ro` | 読み取り専用                                                                |
| `z`  | SELinux: バインドマウントのコンテンツを複数コンテナ間で共有                 |
| `Z`  | SELinux: バインドマウントのコンテンツをプライベートに（他コンテナと非共有） |

> **注意**: 相対パスは Compose ファイルのディレクトリからの相対パスとして解釈されます。  
> 名前付きボリュームとの混同を避けるため、相対パスは必ず `./` または `../` で始めてください。

> **補足**: 短縮記法でのバインドマウントでは、ホスト側のパスが存在しない場合にディレクトリが自動作成されます（後方互換動作）。  
> これを防ぐには詳細記法で `create_host_path: false` を指定します。

---

## 詳細記法（Long syntax）

短縮記法では表現できない詳細な設定ができます。

```yaml
services:
  backend:
    volumes:
      - type: volume
        source: db-data
        target: /data
        volume:
          nocopy: true
          subpath: sub
      - type: bind
        source: /var/run/postgres/postgres.sock
        target: /var/run/postgres/postgres.sock

volumes:
  db-data:
```

### 共通フィールド

| フィールド  | 説明                                                                         |
| ----------- | ---------------------------------------------------------------------------- |
| `type`      | マウント種別: `volume` / `bind` / `tmpfs` / `image` / `npipe` / `cluster`    |
| `source`    | マウント元。バインドマウントはホストパス、ボリュームは名前、`tmpfs` では不要 |
| `target`    | コンテナ内のマウント先パス                                                   |
| `read_only` | `true` にすると読み取り専用（デフォルト: `false`）                           |

### `bind` 固有のフィールド（`bind:`）

| フィールド         | 説明                                                                      |
| ------------------ | ------------------------------------------------------------------------- |
| `propagation`      | バインドマウントの伝播モード                                              |
| `create_host_path` | `true` の場合、ホスト側パスが存在しなければ自動作成（デフォルト: `true`） |
| `selinux`          | SELinux の再ラベルオプション: `z`（共有）または `Z`（プライベート）       |

```yaml
- type: bind
  source: ./data
  target: /app/data
  bind:
    create_host_path: false
```

### `volume` 固有のフィールド（`volume:`）

| フィールド | 説明                                                                  |
| ---------- | --------------------------------------------------------------------- |
| `nocopy`   | `true` にすると、ボリューム作成時にコンテナからのデータコピーを無効化 |
| `subpath`  | ボリュームのルートではなく、ボリューム内の特定サブパスをマウント      |

```yaml
- type: volume
  source: mydata
  target: /app/data
  volume:
    subpath: app/config
    nocopy: true
```

### `tmpfs` 固有のフィールド（`tmpfs:`）

| フィールド | 説明                                                                   |
| ---------- | ---------------------------------------------------------------------- |
| `size`     | tmpfs のサイズ（バイト数または単位付き文字列）                         |
| `mode`     | ファイルシステムのパーミッション（8 進数）。Docker Compose 2.14.0 以降 |

```yaml
- type: tmpfs
  target: /tmp
  tmpfs:
    size: "64m"
    mode: 0755
```

---

## 名前付きボリュームの宣言

複数のサービスでボリュームを共有する場合は、トップレベルの `volumes` セクションで名前付きボリュームを宣言します。

```yaml
services:
  db:
    image: postgres:18
    volumes:
      - db-data:/var/lib/postgresql/data
  backup:
    image: backup-tool
    volumes:
      - db-data:/data:ro   # 同じボリュームを別サービスが読み取り専用でマウント

volumes:
  db-data:   # 名前付きボリュームの宣言
```

---

## バインドマウント vs 名前付きボリュームの使い分け

|                | バインドマウント                             | 名前付きボリューム              |
| -------------- | -------------------------------------------- | ------------------------------- |
| ソース         | ホスト側の特定パス                           | Docker が管理する仮想ストレージ |
| 主な用途       | 開発時のソースコード同期、設定ファイルの注入 | DB データなどの永続化           |
| ポータビリティ | 低い（パスがホスト依存）                     | 高い                            |
| パフォーマンス | ホスト依存                                   | 一般的にボリュームの方が高速    |

---

## まとめ

| ポイント           | 内容                                           |
| ------------------ | ---------------------------------------------- |
| 短縮記法           | `VOLUME:CONTAINER_PATH[:ACCESS_MODE]`          |
| 詳細記法           | `type` / `source` / `target` などで細かく制御  |
| 名前付きボリューム | 複数サービス間で共有。トップレベルで宣言が必要 |
| バインドマウント   | ホストのパスをコンテナに直接マウント           |
| `tmpfs`            | メモリ上の一時ファイルシステム                 |
