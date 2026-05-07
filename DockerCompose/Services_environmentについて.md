# Docker Compose — `services.environment` について

> 参考: [公式ドキュメント — Define services in Docker Compose # environment](https://docs.docker.com/reference/compose-file/services/#environment)

---

## 概要

`environment` は、サービスのコンテナに設定する環境変数を定義します。  
**マップ形式** と **配列形式** の 2 種類の書き方に対応しています。

---

## マップ形式

```yaml
environment:
  RACK_ENV: development
  SHOW: "true"
  USER_INPUT:
```

---

## 配列形式

```yaml
environment:
  - RACK_ENV=development
  - SHOW=true
  - USER_INPUT
```

---

## 値なし（キーのみ）の指定

値を持たないキーのみの記述（`USER_INPUT:` または `- USER_INPUT`）は、**ホストマシンの環境変数から値を引き継ぐ** ことを意図しています。  
ホスト側に該当する環境変数が存在しない場合、その変数はコンテナの環境に設定されません（unset になります）。

```yaml
environment:
  - DATABASE_URL   # ホスト側の DATABASE_URL の値をそのまま使用
```

---

## ブール値の扱い

YAML パーサーによって `true` / `false` / `yes` / `no` が `True` / `False` に変換されることを防ぐため、**クォートで囲む**ことを推奨します。

```yaml
environment:
  DEBUG: "true"   # 推奨
  DEBUG: true     # YAML によっては True に変換されてしまう可能性がある
```

---

## `env_file` との関係

`env_file` と `environment` を両方設定した場合、**`environment` の値が優先されます**（値が空や未定義でも同様）。

```yaml
services:
  app:
    env_file:
      - ./default.env
    environment:
      APP_ENV: production   # default.env に APP_ENV があっても上書きされる
```

---

## 変数の展開（Interpolation）

`environment` の値には、Compose の変数展開（`${VAR}` 構文）を使用できます。  
値は `.env` ファイルやシェルの環境変数から動的に参照できます。

```yaml
environment:
  TAG: ${APP_VERSION:-latest}   # APP_VERSION が未設定なら "latest"
```

---

## `env_file` との使い分け

| 方法          | 向いているケース                                                    |
| ------------- | ------------------------------------------------------------------- |
| `environment` | 数が少ない変数や、明示的に上書きしたい変数                          |
| `env_file`    | 変数が多い場合、または機密情報を Compose ファイルに書きたくない場合 |

---

## まとめ

| ポイント | 内容                                                     |
| -------- | -------------------------------------------------------- |
| 記法     | マップ形式・配列形式の両方に対応                         |
| 値なし   | ホストの同名環境変数を引き継ぐ（存在しない場合は unset） |
| ブール値 | クォートで囲んでおくと安全                               |
| 優先順位 | `environment` > `env_file`                               |
