# Docker Compose — `services.image` について

> 参考: [公式ドキュメント — Define services in Docker Compose # image](https://docs.docker.com/reference/compose-file/services/#image)

---

## 概要

`image` は、サービスのコンテナを起動する際に使用する Docker イメージを指定するフィールドです。

値は **OCI Image Spec の addressable image format** に従う必要があります。

```
[<registry>/][<project>/]<image>[:<tag>|@<digest>]
```

---

## 記述例

```yaml
services:
  app:
    image: redis
```

以下はすべて有効な記述です。

```yaml
image: redis
image: redis:5
image: redis@sha256:0ed5d5928d4737458944eb604cc8509e245c3e19d02ad83935398bc4b991aac7
image: library/redis
image: docker.io/library/redis
image: my_private.registry:5000/redis
```

---

## 書式の各要素

| 要素       | 必須     | 説明                                                                   |
| ---------- | -------- | ---------------------------------------------------------------------- |
| `registry` | 任意     | レジストリのホスト名。省略時は Docker Hub（`docker.io`）が使用される   |
| `project`  | 任意     | 名前空間（Docker Hub では `library` など）。省略時はデフォルト名前空間 |
| `image`    | **必須** | イメージ名                                                             |
| `tag`      | 任意     | バージョンタグ。省略時は `latest` として扱われる                       |
| `digest`   | 任意     | `@sha256:...` 形式でイメージを固定できる。`tag` と同時には指定できない |

---

## タグ vs ダイジェスト

### タグ指定

```yaml
image: redis:7.2
```

- タグは可変です。同じタグ名でも、レジストリ側でイメージが更新されることがあります。
- 再現性を完全に保証したい場合はダイジェスト指定を推奨します。

### ダイジェスト指定

```yaml
image: redis@sha256:0ed5d5928d4737458944eb604cc8509e245c3e19d02ad83935398bc4b991aac7
```

- イメージの内容をハッシュ値で一意に固定します。
- CI/CD や本番環境で「全く同じイメージを使う」ことを保証したい場合に有効です。

---

## `image` を省略できる場合

`image` フィールドは、`build` セクションが宣言されている場合に限り省略できます。

```yaml
services:
  app:
    build: ./app   # image を省略可能
```

`build` セクションがない場合、`image` は必須です。省略すると Compose はエラーになります。

---

## イメージの取得タイミング（`pull_policy`）

`image` が指定されていてもプラットフォーム上に存在しない場合、Compose は `pull_policy` の設定に従ってイメージを取得しようとします。

| `pull_policy` の値      | 説明                                                                |
| ----------------------- | ------------------------------------------------------------------- |
| `missing`（デフォルト） | ローカルキャッシュになければ取得する。`latest` タグは常に取得される |
| `always`                | 常にレジストリから取得する                                          |
| `never`                 | 取得しない。キャッシュになければエラー                              |
| `build`                 | `build` セクションを使ってイメージをビルドする                      |
| `daily`                 | 最終取得から 24 時間以上経過していれば更新を確認する                |
| `weekly`                | 最終取得から 7 日以上経過していれば更新を確認する                   |
| `every_<duration>`      | 指定した期間（例: `every_12h`）経過していれば更新を確認する         |

```yaml
services:
  app:
    image: nginx
    pull_policy: always
```

---

## プライベートレジストリの利用

Docker Hub 以外のプライベートレジストリを使う場合は、レジストリホストを含めたフルパスで指定します。

```yaml
services:
  app:
    image: my_private.registry:5000/myapp:1.0
```

認証が必要な場合は、`docker login` で事前にログインしておくか、CI/CD 環境では `~/.docker/config.json` に認証情報を設定します。

---

## まとめ

| ポイント         | 内容                                         |
| ---------------- | -------------------------------------------- |
| フォーマット     | `[registry/][project/]image[:tag\|@digest]`  |
| タグ省略時       | `latest` として扱われる                      |
| ダイジェスト指定 | イメージ内容を完全に固定できる               |
| `image` 省略     | `build` セクションがある場合のみ可能         |
| 取得ポリシー     | `pull_policy` で制御。デフォルトは `missing` |
