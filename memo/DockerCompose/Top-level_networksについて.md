# Docker Compose — トップレベル `networks` について

> 参考: [公式ドキュメント — Define and manage networks in Docker Compose](https://docs.docker.com/reference/compose-file/networks/)  
> 参考: [公式ドキュメント — Network drivers](https://docs.docker.com/engine/network/drivers/)

---

## 概要

ネットワークはサービス間の通信を可能にします。トップレベルの `networks` セクションは、複数のサービスで再利用できる**名前付きネットワーク**を宣言・設定するための要素です。

デフォルトでは、Compose はアプリ全体に対して暗黙の `default` ネットワークを 1 つ作成します。同一 Compose ファイル内のサービスはこのネットワークに自動接続され、**サービス名で名前解決**できます。

---

## 基本的な使い方

```yaml
services:
  frontend:
    image: example/webapp
    networks:
      - front-tier
      - back-tier

networks:
  front-tier:
  back-tier:
```

複数のネットワークを定義することで、サービス間の通信を分離できます。

```yaml
services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres:18
    networks:
      - backend

networks:
  frontend:
  backend:
```

この例では `proxy` と `db` は共通のネットワークを持たないため通信できません。`app` のみが両方と通信できます。

---

## デフォルトネットワーク

`networks` を明示しない場合、Compose は暗黙の `default` ネットワークを作成し、すべてのサービスを接続します。

```yaml
# 省略した場合…
services:
  some-service:
    image: foo

# …以下と同等
services:
  some-service:
    image: foo
    networks:
      default: {}
networks:
  default: {}
```

`default` ネットワークもカスタマイズできます。

```yaml
networks:
  default:
    name: my-app-network
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
```

---

## 属性一覧

### `driver`

使用するネットワークドライバを指定します。指定したドライバがプラットフォームで利用できない場合、Compose はエラーを返します。

```yaml
networks:
  mynet:
    driver: bridge
```

#### ネットワークドライバの種類

| ドライバ  | 説明                                                                                                     | 主な用途                             |
| --------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `bridge`  | **デフォルト**。同一ホスト上のコンテナ間通信用の仮想ネットワーク。何も指定しなければこれが使われる       | 同一ホスト内のサービス間通信         |
| `host`    | コンテナとホストのネットワーク分離をなくす。コンテナがホストのネットワークインターフェースを直接使用する | 高いネットワーク性能が必要な場合     |
| `overlay` | 複数の Docker ホスト（デーモン）をまたいで通信する。Docker Swarm で使用する                              | 複数ホスト間のサービス通信（Swarm）  |
| `macvlan` | コンテナに MAC アドレスを割り当て、物理デバイスとして見せる                                              | レガシーアプリや物理ネットワーク接続 |
| `ipvlan`  | MAC アドレスを割り当てずに IPv4/IPv6 を制御する。`macvlan` の MAC 数制限がある環境での代替               | MAC アドレスに制限がある環境         |
| `none`    | コンテナを完全にネットワーク分離する                                                                     | ネットワーク不要なバッチ処理など     |

> **ポイント**: Compose での開発用途では `bridge`（省略時のデフォルト）で十分です。

### `driver_opts`

ドライバに渡すオプションをキーバリューペアで指定します。オプションはドライバに依存します。

```yaml
networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
```

### `attachable`

`true` に設定すると、サービスコンテナ以外のスタンドアロンコンテナもこのネットワークに接続できます。`overlay` ドライバと組み合わせて使います。

```yaml
networks:
  mynet:
    driver: overlay
    attachable: true
```

### `internal`

`true` に設定すると、外部との通信を遮断した**完全隔離ネットワーク**を作成します。

```yaml
networks:
  private:
    internal: true
```

### `enable_ipv4` / `enable_ipv6`

IPv4 / IPv6 アドレスの割り当てを制御します（Docker Compose 2.33.1 以降）。

```yaml
networks:
  ip6net:
    enable_ipv4: false
    enable_ipv6: true
```

### `external`

`true` に設定すると、このネットワークは Compose の管理外の**既存ネットワーク**として扱われます。

- Compose はネットワークを作成しません。ネットワークが存在しない場合はエラーになります。
- `name` 以外の属性は無視されます（他の属性を記述すると Compose ファイルが無効とみなされます）。

```yaml
services:
  proxy:
    image: example/proxy
    networks:
      - outside
      - default
  app:
    image: example/app
    networks:
      - default

networks:
  outside:
    external: true
```

### `ipam`

IP アドレス管理（IPAM）の設定をカスタマイズします。サブネットや IP 範囲、ゲートウェイを明示的に指定したい場合に使います。

```yaml
networks:
  mynet:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          ip_range: 172.28.5.0/24
          gateway: 172.28.5.254
          aux_addresses:
            host1: 172.28.1.5
            host2: 172.28.1.6
      options:
        foo: bar
```

| フィールド      | 説明                                           |
| --------------- | ---------------------------------------------- |
| `driver`        | カスタム IPAM ドライバ（省略時はデフォルト）   |
| `subnet`        | ネットワークセグメントを CIDR 形式で指定       |
| `ip_range`      | コンテナへの IP 割り当て範囲                   |
| `gateway`       | サブネットのゲートウェイ IP                    |
| `aux_addresses` | ドライバが使用する補助 IP アドレスのマッピング |

### `labels`

ネットワークにメタデータを付与します。マップ形式・配列形式の両方に対応しています。  
ラベルの衝突を避けるため、**逆 DNS 記法**を使うことが推奨されています。

```yaml
networks:
  mynet:
    labels:
      com.example.description: "Financial transaction network"
      com.example.department: "Finance"
```

> Compose は自動的に `com.docker.compose.project` と `com.docker.compose.network` ラベルを付与します。

### `name`

ネットワークにカスタム名を設定します。名前はプロジェクト名でスコープされず、**そのまま使用されます**。

```yaml
networks:
  network1:
    name: my-app-net
```

`external: true` と組み合わせることで、Compose ファイル内での参照名と実際のネットワーク名を分離できます。

```yaml
networks:
  network1:
    external: true
    name: "${NETWORK_ID}"
```

---

## 属性まとめ

| 属性          | 説明                                                                       |
| ------------- | -------------------------------------------------------------------------- |
| `driver`      | 使用するネットワークドライバを指定する（デフォルト: `bridge`）             |
| `driver_opts` | ドライバに渡すオプションをキーバリューペアで指定する                       |
| `attachable`  | スタンドアロンコンテナからの接続を許可する（`overlay` と組み合わせて使う） |
| `internal`    | 外部との通信を遮断した隔離ネットワークを作成する                           |
| `enable_ipv4` | IPv4 アドレス割り当ての有効/無効を制御する（2.33.1 以降）                  |
| `enable_ipv6` | IPv6 アドレス割り当てを有効にする                                          |
| `external`    | Compose 管理外の既存ネットワークを参照する                                 |
| `ipam`        | サブネット・IP 範囲・ゲートウェイなどを手動設定する                        |
| `labels`      | ネットワークにメタデータ（ラベル）を付与する                               |
| `name`        | ネットワークのカスタム名を設定する（プロジェクト名でスコープされない）     |

---

## `services.networks` との関係

| 要素                    | 役割                                               |
| ----------------------- | -------------------------------------------------- |
| トップレベル `networks` | 名前付きネットワークを宣言・設定する               |
| `services.networks`     | 各サービスがどのネットワークに接続するかを指定する |

名前付きネットワークを使うには、トップレベルで宣言した上で、各サービスの `networks` 属性で参照する必要があります。
