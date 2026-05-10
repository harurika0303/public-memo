# Laravel 概要

> 参考:
> - [Installation — Laravel 12.x](https://laravel.com/docs/12.x)
> - [Request Lifecycle — Laravel 12.x](https://laravel.com/docs/12.x/lifecycle)
> - [Directory Structure — Laravel 12.x](https://laravel.com/docs/12.x/structure)

---

## Laravel とは

Laravel は **PHP で書かれた Web アプリケーションフレームワーク**です。表現力豊かでエレガントな構文を特徴とし、ルーティング・認証・データベース操作・キュー・テストなど、本格的な Web アプリケーションに必要な機能を標準で備えています。

> "Laravel strives to provide an amazing developer experience while providing powerful features such as thorough dependency injection, an expressive database abstraction layer, queues and scheduled jobs, unit and integration testing, and more."
> — 公式ドキュメント

### Laravel の 4 つの特徴

| 特徴                         | 説明                                                                                     |
| ---------------------------- | ---------------------------------------------------------------------------------------- |
| **プログレッシブ（段階的）** | 初心者から上級者まで段階的に成長できる。豊富なドキュメント・Laracasts 動画が揃っている   |
| **スケーラブル**             | Redis などの分散キャッシュと相性がよく、水平スケールが容易。月間数億リクエストの実績あり |
| **AI 対応**                  | 規約ベースの構造が明確なため、Cursor・Claude Code などの AI ツールと相性がよい           |
| **コミュニティ**             | PHP エコシステムのベストパッケージを統合。世界中の開発者が貢献している                   |

---

## 2 つの主な使い方

### 1. フルスタックフレームワークとして使う

Laravel でルーティング・コントローラ・ビューをすべて処理します。フロントエンドは [Blade テンプレート](https://laravel.com/docs/12.x/blade)（PHP ベースのテンプレートエンジン）か、[Inertia.js](https://inertiajs.com/)（React/Vue と組み合わせる SPA ハイブリッド）で構築します。

```
ブラウザ → Laravel（ルーティング・ビジネスロジック・Blade テンプレート）→ HTML レスポンス
```

関連する技術：Blade, Livewire, Inertia.js, Vite

### 2. API バックエンドとして使う

Next.js・React・Vue などのフロントエンドや、モバイルアプリのバックエンド API として Laravel を使います。認証は [Sanctum](https://laravel.com/docs/12.x/sanctum)（トークン認証）で行います。

```
React / Next.js / モバイルアプリ → Laravel API（認証・データ操作・通知など）→ JSON レスポンス
```

---

## リクエストのライフサイクル

すべてのリクエストは `public/index.php` に到達し、以下の流れで処理されます。

```
1. public/index.php
   └── Composer のオートローダー読み込み
   └── bootstrap/app.php でアプリケーション（サービスコンテナ）を生成

2. HTTP Kernel（Illuminate\Foundation\Http\Kernel）
   └── ブートストラップ（エラーハンドリング・ロギング・環境設定など）
   └── グローバルミドルウェアの適用（セッション・CSRF・メンテナンスモードなど）

3. サービスプロバイダの登録・起動
   └── データベース・キュー・バリデーション・ルーティングなどの初期化

4. ルーター
   └── リクエストを適切なコントローラ・クロージャにディスパッチ
   └── ルート固有のミドルウェアを適用

5. コントローラ / クロージャの実行
   └── レスポンスを返す

6. ミドルウェアを逆順に通過してレスポンスを送信
```

---

## ディレクトリ構造

```
プロジェクトルート/
├── app/                   ← アプリケーションの中心コード
│   ├── Http/              ← コントローラ・ミドルウェア・フォームリクエスト
│   ├── Models/            ← Eloquent モデル
│   ├── Providers/         ← サービスプロバイダ
│   ├── Jobs/              ← キュージョブ
│   ├── Events/            ← イベントクラス
│   ├── Listeners/         ← イベントリスナー
│   ├── Mail/              ← メールクラス
│   ├── Notifications/     ← 通知クラス
│   └── Policies/          ← 認可ポリシー
├── bootstrap/             ← フレームワーク起動ファイル・キャッシュ
├── config/                ← 設定ファイル（app.php, database.php など）
├── database/
│   ├── migrations/        ← データベースマイグレーション
│   ├── factories/         ← テスト用ファクトリ
│   └── seeders/           ← シードデータ
├── public/                ← Web サーバーの公開ルート（index.php・静的ファイル）
├── resources/
│   ├── views/             ← Blade テンプレート
│   └── css/ / js/         ← フロントエンドの未コンパイルソース
├── routes/
│   ├── web.php            ← Web ルート（セッション・CSRF 保護あり）
│   ├── console.php        ← Artisan コマンド定義
│   └── api.php            ← API ルート（ステートレス）※ オプション
├── storage/               ← ログ・コンパイル済みテンプレート・キャッシュ
├── tests/                 ← テストファイル（Pest / PHPUnit）
├── vendor/                ← Composer 依存パッケージ
├── .env                   ← 環境変数（Git 管理外）
└── artisan                ← Artisan CLI エントリーポイント
```

### 重要なファイル・ディレクトリの説明

| パス                    | 説明                                                  |
| ----------------------- | ----------------------------------------------------- |
| `public/index.php`      | すべてのリクエストの入口。Web サーバーはここに向ける  |
| `app/Http/Controllers/` | コントローラを置く場所                                |
| `app/Models/`           | Eloquent モデル                                       |
| `routes/web.php`        | ブラウザ向けのルート定義                              |
| `routes/api.php`        | API ルート（`/api/` プレフィックスが付く）            |
| `resources/views/`      | Blade テンプレート（`.blade.php`）                    |
| `database/migrations/`  | テーブル定義の変更履歴                                |
| `config/`               | 各種設定ファイル                                      |
| `.env`                  | 接続情報・APIキーなど環境ごとの設定（コミットしない） |

---

## 主要な機能・コンポーネント

| 機能                   | 説明                                                                   | ドキュメント                                                   |
| ---------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------- |
| **ルーティング**       | URL とコントローラを紐付ける。クロージャ・リソースルートも書ける       | [Routing](https://laravel.com/docs/12.x/routing)               |
| **Eloquent ORM**       | ActiveRecord パターンのデータベース抽象レイヤー                        | [Eloquent](https://laravel.com/docs/12.x/eloquent)             |
| **Blade テンプレート** | PHP ベースのテンプレートエンジン。レイアウト継承・コンポーネントに対応 | [Blade](https://laravel.com/docs/12.x/blade)                   |
| **マイグレーション**   | データベーススキーマをコードで管理する                                 | [Migrations](https://laravel.com/docs/12.x/migrations)         |
| **認証**               | セッション認証・トークン認証（Sanctum）に対応                          | [Authentication](https://laravel.com/docs/12.x/authentication) |
| **ミドルウェア**       | リクエスト・レスポンスのフィルタリング処理                             | [Middleware](https://laravel.com/docs/12.x/middleware)         |
| **サービスコンテナ**   | 依存性注入（DI）の仕組み                                               | [Service Container](https://laravel.com/docs/12.x/container)   |
| **サービスプロバイダ** | アプリケーション起動時の初期化処理をまとめる場所                       | [Service Providers](https://laravel.com/docs/12.x/providers)   |
| **Artisan CLI**        | `php artisan` で使えるコマンドラインツール                             | [Artisan](https://laravel.com/docs/12.x/artisan)               |
| **キュー**             | ジョブをバックグラウンドで非同期処理する                               | [Queues](https://laravel.com/docs/12.x/queues)                 |
| **バリデーション**     | リクエストデータの入力検証                                             | [Validation](https://laravel.com/docs/12.x/validation)         |
| **テスト**             | Pest / PHPUnit によるユニット・フィーチャーテスト                      | [Testing](https://laravel.com/docs/12.x/testing)               |
| **通知**               | メール・Slack・SMS など複数チャンネルへの通知                          | [Notifications](https://laravel.com/docs/12.x/notifications)   |
| **イベント**           | アプリケーション内イベントの発行・購読                                 | [Events](https://laravel.com/docs/12.x/events)                 |

---

## 主要な公式パッケージ（エコシステム）

| パッケージ    | 説明                                                           |
| ------------- | -------------------------------------------------------------- |
| **Sanctum**   | SPA・モバイルアプリ向けのシンプルなトークン認証                |
| **Socialite** | GitHub・Google などの OAuth ログイン                           |
| **Cashier**   | Stripe/Paddle による定期課金・サブスクリプション管理           |
| **Horizon**   | Redis キューのダッシュボード・監視                             |
| **Telescope** | デバッグアシスタント（リクエスト・クエリ・ジョブなどを可視化） |
| **Pulse**     | アプリケーションのパフォーマンスモニタリング                   |
| **Dusk**      | ブラウザ自動テスト                                             |
| **Scout**     | Eloquent モデルのフルテキスト検索                              |
| **Sail**      | Docker を使った Laravel 開発環境                               |
| **Reverb**    | Laravel 公式の WebSocket サーバー                              |
| **Pennant**   | フィーチャーフラグ管理                                         |
| **Pint**      | PHP コードスタイルフィクサー（Laravel 公式）                   |
| **Octane**    | FrankenPHP/Swoole でアプリを高速化                             |

---

## インストール（最小手順）

### 前提条件

- PHP 8.2 以上
- Composer
- Node.js / npm（フロントエンドアセットをコンパイルする場合）

### 新規プロジェクト作成

```bash
# Laravel インストーラを使う場合
composer global require laravel/installer
laravel new example-app

# または Composer 直接
composer create-project laravel/laravel example-app
```

### 開発サーバー起動

```bash
cd example-app
npm install && npm run build
composer run dev      # PHP サーバー・キューワーカー・Vite を一括起動
```

`http://localhost:8000` でアクセスできます。

### 環境設定（`.env`）

```bash
# データベース設定（デフォルトは SQLite）
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=

# マイグレーション実行
php artisan migrate
```

---

## Artisan コマンド（よく使うもの）

```bash
php artisan serve                   # 開発サーバー起動
php artisan migrate                 # マイグレーション実行
php artisan migrate:rollback        # 直前のマイグレーションを戻す
php artisan make:model Post         # モデル作成
php artisan make:controller PostController  # コントローラ作成
php artisan make:migration create_posts_table  # マイグレーションファイル作成
php artisan make:request StorePostRequest      # フォームリクエスト作成
php artisan route:list              # 定義済みルートの一覧を表示
php artisan tinker                  # インタラクティブな REPL（動作確認に便利）
php artisan test                    # テスト実行
php artisan config:cache            # 設定ファイルをキャッシュ（本番用）
php artisan list make               # make: コマンドの一覧を表示
```
