# Composer について

> 参考:
> - [Introduction — Composer](https://getcomposer.org/doc/00-intro.md)
> - [Basic usage — Composer](https://getcomposer.org/doc/01-basic-usage.md)

---

## Composer とは

Composer は **PHP の依存パッケージ管理ツール**です。JavaScript の npm、Python の pip に相当します。

- プロジェクトが依存するライブラリを `composer.json` に宣言する
- `composer install` でライブラリを自動ダウンロード・インストールする
- ライブラリが依存するさらに別のライブラリも芋づる式に解決してくれる

> Composer はパッケージをプロジェクト単位で管理します（グローバルではなく `vendor/` ディレクトリ内にインストール）。

---

## 基本的な使い方

### 新しいライブラリを追加する

```bash
composer require monolog/monolog
```

実行すると：
1. [Packagist.org](https://packagist.org)（PHP の公開パッケージリポジトリ）からパッケージを検索
2. `vendor/` ディレクトリにダウンロード・インストール
3. `composer.json` の `require` にパッケージ名とバージョンが追記される
4. `composer.lock` にインストールした正確なバージョンが記録される

### 既存プロジェクトの依存関係をインストールする

```bash
# composer.lock に記録されたバージョンを正確に再現してインストール
composer install
```

Git からクローンしたプロジェクトを動かすときに使います。`vendor/` は `.gitignore` でGit管理外にするため、クローン後に必ず実行が必要です。

### バージョンをアップデートする

```bash
# すべての依存パッケージを composer.json の制約内で最新にする
composer update

# 特定のパッケージだけ更新する
composer update monolog/monolog
```

---

## composer.json

プロジェクトの依存関係とメタ情報を記述するファイルです。

```json
{
    "require": {
        "php": "^8.2",
        "laravel/framework": "^12.0",
        "monolog/monolog": "2.0.*"
    },
    "require-dev": {
        "phpunit/phpunit": "^11.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    }
}
```

| キー          | 説明                                                                   |
| ------------- | ---------------------------------------------------------------------- |
| `require`     | 本番環境も含めて必要なパッケージ                                       |
| `require-dev` | 開発・テスト時のみ必要なパッケージ（本番では `--no-dev` で除外できる） |
| `autoload`    | オートローダーの設定（後述）                                           |

### バージョン制約の書き方

| 表記    | 意味                                          |
| ------- | --------------------------------------------- |
| `2.0.*` | `2.0.x`（2.0系の最新）                        |
| `^12.0` | `>= 12.0, < 13.0`（メジャーバージョンを固定） |
| `~2.0`  | `>= 2.0, < 3.0`                               |
| `2.0.5` | 完全固定                                      |

---

## composer.lock

`composer install` / `composer update` を実行すると自動生成されるファイルです。インストールした**正確なバージョン**が記録されます。

```
composer.json  → 「2.0系を使いたい」という意図を書く
composer.lock  → 「実際に 2.0.3 をインストールした」という事実を記録する
```

- **Git にコミットする**：チームメンバー・CI・本番サーバーで完全に同じバージョンを再現できる
- `composer install` は `composer.lock` を優先して読む（バージョンがぶれない）
- `composer update` は `composer.json` の制約内で最新版を取得し `composer.lock` を更新する

---

## オートローダー

### オートローダーがないと何が困るか

PHP はクラスを使う前に `require` でファイルを手動読み込みする必要があります。

```php
// オートローダーなしの場合、すべて手動で読み込む必要がある
require __DIR__ . '/app/Models/User.php';
require __DIR__ . '/app/Http/Controllers/UserController.php';
require __DIR__ . '/vendor/monolog/monolog/src/Monolog/Logger.php';
// ...何十・何百ファイルも書くことになる
```

### Composer のオートローダーで解決する

`composer install` を実行すると `vendor/autoload.php` が自動生成されます。このファイルを 1 回読み込むだけで、**クラス名からファイルパスを自動解決するマッピング**が登録されます。

```php
// これ 1 行だけ書けば以降はすべてのクラスが自動で読み込まれる
require __DIR__ . '/vendor/autoload.php';

// require なしでいきなり使える
$log = new Monolog\Logger('name');
$user = new App\Models\User();
```

### 仕組み：PSR-4 の名前空間とディレクトリのマッピング

`composer.json` の `autoload` にルールを書きます。

```json
{
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        }
    }
}
```

これは「`App\` で始まるクラスは `app/` ディレクトリの中にある」というマッピングです。

```
クラス名                              → ファイルパス
App\Models\User                      → app/Models/User.php
App\Http\Controllers\UserController  → app/Http/Controllers/UserController.php
```

クラスを `new App\Models\User()` と書いた瞬間に PHP がオートローダーを呼び出し、`app/Models/User.php` を自動で `require` してくれます。

ルールを変更した後は以下のコマンドでオートローダーを再生成します。

```bash
composer dump-autoload
```

---

## Laravel との関係

### プロジェクト作成

```bash
composer create-project laravel/laravel example-app
```

Packagist から `laravel/laravel` パッケージを取得してプロジェクトを作成します。

```bash
composer global require laravel/installer
```

`global` オプションをつけると、プロジェクト外（システム全体）にインストールします。これにより `laravel new` コマンドがどこからでも使えるようになります。

### `public/index.php` でのオートローダー読み込み

Laravel のリクエストライフサイクルの最初のステップがこれです。

```php
// public/index.php の先頭
require __DIR__.'/../vendor/autoload.php';
```

「以降のすべてのクラス（Laravel 本体・自分が書いたコード・サードパーティライブラリ）を名前空間で自動解決できるように準備する」処理です。これが終わって初めて `new Application()` などが呼び出せる状態になります。

### vendor/ ディレクトリ

Composer がインストールしたすべての依存パッケージが入るディレクトリです。

```
vendor/
├── laravel/           ← Laravel 本体
├── monolog/           ← ログライブラリ（Laravel の依存）
├── symfony/           ← Symfony コンポーネント（Laravel の依存）
├── ...
└── autoload.php       ← ここを読み込むだけですべてのクラスが使える
```

`vendor/` はサイズが大きく再現可能なため、`.gitignore` でGit管理外にします。クローン後に `composer install` で復元します。

---

## よく使うコマンドまとめ

```bash
composer install                    # composer.lock に従って依存関係をインストール
composer update                     # 依存関係を最新バージョンに更新
composer require パッケージ名        # パッケージを追加
composer require --dev パッケージ名  # 開発用パッケージを追加
composer remove パッケージ名         # パッケージを削除
composer dump-autoload              # オートローダーを再生成
composer show                       # インストール済みパッケージ一覧を表示
composer outdated                   # 更新可能なパッケージを表示
```
