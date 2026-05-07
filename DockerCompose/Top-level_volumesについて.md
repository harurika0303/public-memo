# Docker Compose — トップレベル `volumes` について

> 参考: [公式ドキュメント — Define and manage volumes in Docker Compose](https://docs.docker.com/reference/compose-file/volumes/)

---

## 概要

トップレベルの `volumes` セクションは、複数のサービスで再利用できる**名前付きボリューム（Named Volume）**を宣言するための要素です。

ボリュームはコンテナエンジンが管理する永続データストアです。Compose は、サービスがボリュームをマウントするための中立的な手段と、インフラに割り当てるための設定パラメータを提供します。

---

## 基本的な使い方

```yaml
services:
  backend:
    image: example/database
    volumes:
      - db-data:/etc/data

  backup:
    image: backup-service
    volumes:
      - db-data:/var/lib/backup/data

volumes:
  db-data:
```

- `docker compose up` 実行時にボリュームが存在しなければ自動作成されます。
- すでに存在する場合はそのまま使用されます（Compose の外で手動削除された場合は再作成されます）。
- 複数のサービスからボリュームを使う場合は、各サービスの `volumes` 属性で明示的に指定する必要があります。

---

## 属性一覧

トップレベル `volumes` のエントリは空のままにできます（その場合はコンテナエンジンのデフォルト設定が使用されます）。

### `driver`

使用するボリュームドライバを指定します。指定したドライバが利用できない場合、Compose はエラーを返してデプロイを中断します。

```yaml
volumes:
  db-data:
    driver: local
```

#### ボリュームドライバとは

ボリュームドライバは、**ボリュームのデータをどこに・どのように保存するか**を決定するプラグインです。  
アプリケーションのロジックを変えずに、ストレージの実装だけを切り替えられる点が特徴です。

| ドライバ                             | 説明                                                                                       |
| ------------------------------------ | ------------------------------------------------------------------------------------------ |
| `local`（デフォルト）                | Docker ホストのローカルファイルシステムに保存する。何も指定しなければこれが使われる        |
| `local`（NFS オプション付き）        | `driver_opts` で NFS マウントを設定する。`local` ドライバ自体を NFS クライアントとして使う |
| `local`（CIFS/Samba オプション付き） | `driver_opts` で Samba 共有をマウントする                                                  |
| サードパーティプラグイン             | `rclone` など、クラウドストレージや外部システムに接続するプラグインをインストールして使う  |

> **ポイント**: 開発環境では `local`（省略時のデフォルト）で十分です。  
> 本番環境でマシン間のデータ共有が必要な場合に NFS や外部ドライバを検討します。

### `driver_opts`

ドライバに渡すオプションをキーバリューペアで指定します。オプションはドライバに依存します。

#### `local` ドライバ（デフォルト）

`driver` を省略した場合と同じです。Docker ホスト上の `/var/lib/docker/volumes/` 以下にデータが保存されます。

```yaml
volumes:
  db-data:
    # driver: local と同じ（省略可）
```

#### `local` ドライバ + NFS マウント

`local` ドライバに `driver_opts` で NFS の接続情報を渡すことで、NFS サーバーのディレクトリをボリュームとして使用できます。

```yaml
volumes:
  nfs-data:
    driver: local
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```

| オプション | 説明                                                       |
| ---------- | ---------------------------------------------------------- |
| `type`     | マウントの種類。NFS の場合は `"nfs"` または `"nfs4"`       |
| `o`        | マウントオプション。`addr` で NFS サーバーの IP を指定する |
| `device`   | NFS サーバー上のエクスポートパス（先頭の `:` は必須）      |

#### `local` ドライバ + CIFS/Samba マウント

```yaml
volumes:
  samba-data:
    driver: local
    driver_opts:
      type: "cifs"
      device: "//192.168.1.100/share"
      o: "username=user,password=pass,file_mode=0777,dir_mode=0777"
```

#### サードパーティドライバ（例: rclone）

プラグインをインストールすることで、クラウドストレージ（S3、Google Drive など）をボリュームとして使用できます。

```yaml
volumes:
  cloud-data:
    driver: rclone
    driver_opts:
      type: s3
      path: mybucket/mypath
```

> **注意**: サードパーティドライバを使う場合は、事前に `docker plugin install` でプラグインをインストールする必要があります。

### `external`

`true` に設定すると、このボリュームはプラットフォーム上に**すでに存在する外部ボリューム**として扱われます。

- Compose はボリュームを作成しません。ボリュームが存在しない場合はエラーになります。
- `name` 以外の属性は無視されます（他の属性を記述すると Compose ファイルが無効とみなされます）。

```yaml
services:
  backend:
    image: example/database
    volumes:
      - db-data:/etc/data

volumes:
  db-data:
    external: true
```

この例では、`{project_name}_db-data` という名前のボリュームを作成するのではなく、`db-data` という名前の既存ボリュームを検索してマウントします。

### `labels`

ボリュームにメタデータを付与します。マップ形式と配列形式の両方に対応しています。  
ラベルの衝突を避けるため、**逆 DNS 記法**を使うことが推奨されています。

```yaml
volumes:
  db-data:
    labels:
      com.example.description: "Database volume"
      com.example.department: "IT/Ops"
      com.example.label-with-empty-value: ""
```

```yaml
volumes:
  db-data:
    labels:
      - "com.example.description=Database volume"
      - "com.example.department=IT/Ops"
      - "com.example.label-with-empty-value"
```

> **注意**: Compose は自動的に `com.docker.compose.project` と `com.docker.compose.volume` ラベルを付与します。  
> ここで定義したラベルは名前付きボリュームにのみ適用されます。バインドマウントには適用されません。

### `name`

ボリュームにカスタム名を設定します。特殊文字を含む既存ボリュームを参照する場合などに使います。  
名前はスタック名でスコープされず、**そのまま使用されます**。

```yaml
volumes:
  db-data:
    name: "my-app-data"
```

変数展開と組み合わせて、デプロイ時に動的に名前を設定することもできます。

```yaml
# .env に DATABASE_VOLUME=my_volume_001 がある場合
volumes:
  db-data:
    name: ${DATABASE_VOLUME}
```

`external: true` と組み合わせることで、Compose ファイル内での参照名と実際のボリューム名を分離できます。

```yaml
volumes:
  db-data:
    external: true
    name: actual-name-of-volume
```

---

## 属性まとめ

| 属性          | 説明                                                                 |
| ------------- | -------------------------------------------------------------------- |
| `driver`      | 使用するボリュームドライバを指定する                                 |
| `driver_opts` | ドライバに渡すオプションをキーバリューペアで指定する                 |
| `external`    | プラットフォーム上の既存ボリュームを参照する（Compose で管理しない） |
| `labels`      | ボリュームにメタデータ（ラベル）を付与する                           |
| `name`        | ボリュームのカスタム名を設定する（スタック名でスコープされない）     |

---

## `services.volumes` との関係

| 要素                   | 役割                                                           |
| ---------------------- | -------------------------------------------------------------- |
| トップレベル `volumes` | 名前付きボリュームを宣言・設定する                             |
| `services.volumes`     | 各サービスがどのボリュームをどのパスにマウントするかを指定する |

名前付きボリュームを使うには、トップレベルで宣言した上で、各サービスの `volumes` 属性で参照する必要があります。

---

## 名前付きボリュームとバインドマウントの違い

`services.volumes` では、名前付きボリューム以外に**バインドマウント**（ホストの任意ディレクトリを直接指定する方法）も使えます。

### 名前付きボリューム（`driver: local`）

Docker が `/var/lib/docker/volumes/<ボリューム名>/_data/` を管理し、そこにデータを保存します。  
コンテナを削除してもデータは残りますが、ホストから自由に触れる場所ではないため「同期」とは異なります。

```
Docker ホスト
└── /var/lib/docker/volumes/myproject_db-data/_data/  ← Docker が管理
        ↕ マウント
コンテナ内
└── /var/lib/postgresql/data/
```

### バインドマウント（ホストの任意ディレクトリを指定）

`services.volumes` でホストのパスを直接指定する方法です。ホスト側のファイルを編集するとコンテナ内にも即座に反映されるため、「同期」に近い動作をします。

```yaml
services:
  app:
    volumes:
      - ./src:/app/src   # ホストの ./src とコンテナの /app/src が同じ中身
```

```
Docker ホスト
└── ./src/  ← 自分で触れる・編集できる
      ↕ バインドマウント（同期）
コンテナ内
└── /app/src/
```

### 使い分けの目安

| 種類                           | ホストから触れる？         | 主な用途                         |
| ------------------------------ | -------------------------- | -------------------------------- |
| 名前付きボリューム（`local`）  | しにくい（Docker 管理下）  | DB データなどの永続化            |
| バインドマウント（パス指定）   | できる（任意の場所）       | ソースコードの同期・開発時       |

開発時にコードをリアルタイムで反映させたい場合はバインドマウント、DB データを永続化したいだけなら名前付きボリュームというのが典型的な使い分けです。
