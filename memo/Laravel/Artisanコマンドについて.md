# Laravel の Artisan コマンド

> 参考:
> - [Artisan Console — Laravel 12.x](https://laravel.com/docs/12.x/artisan)

---

## Artisan とは

Artisan は **Laravel に組み込まれたCLIツール**です。プロジェクトルートの `artisan` ファイルがエントリーポイントで、`php artisan コマンド名` の形式で実行します。

```bash
php artisan list          # 利用可能なコマンド一覧を表示
php artisan help migrate  # 特定コマンドのヘルプを表示
```

Laravel の開発では、ファイル生成・DB操作・サーバー起動・キャッシュ管理など、あらゆる場面でArtisanコマンドを使います。

---

## 場面別コマンド一覧

### プロジェクト初期セットアップ

```bash
# .env ファイルのアプリケーションキーを生成（初回セットアップ時に必須）
php artisan key:generate

# ストレージへのシンボリックリンクを作成（ファイル公開用）
php artisan storage:link
```

---

### ファイル生成（make:系）

Laravel では `make:` 系コマンドでファイルのひな形を自動生成します。

```bash
# コントローラ
php artisan make:controller PostController
php artisan make:controller PostController --resource   # CRUD メソッド付き
php artisan make:controller PostController --api        # API 用（show 追加、create/edit なし）

# モデル
php artisan make:model Post
php artisan make:model Post -m                          # マイグレーションも同時生成
php artisan make:model Post -mrc                        # マイグレーション + リソースコントローラ + ファクトリも同時生成

# マイグレーション
php artisan make:migration create_posts_table
php artisan make:migration add_status_to_posts_table --table=posts

# シーダー（初期データ投入）
php artisan make:seeder PostSeeder

# ファクトリ（テスト用ダミーデータ生成）
php artisan make:factory PostFactory

# フォームリクエスト（バリデーション）
php artisan make:request StorePostRequest

# ミドルウェア
php artisan make:middleware CheckAge

# イベント / リスナー
php artisan make:event OrderPlaced
php artisan make:listener SendOrderConfirmation

# ジョブ（キュー処理）
php artisan make:job ProcessOrder

# メール
php artisan make:mail OrderShipped

# 通知
php artisan make:notification InvoicePaid

# ポリシー（認可）
php artisan make:policy PostPolicy --model=Post

# コマンド（独自 Artisan コマンド）
php artisan make:command SendEmails
```

---

### データベース操作

```bash
# 未実行のマイグレーションをすべて実行
php artisan migrate

# マイグレーションの実行状況を確認
php artisan migrate:status

# 直前のバッチをロールバック
php artisan migrate:rollback
php artisan migrate:rollback --step=2   # 2バッチ分ロールバック

# すべてロールバック → 再マイグレーション（データは消える）
php artisan migrate:refresh
php artisan migrate:refresh --seed      # シーダーも実行

# テーブルを全削除 → 再マイグレーション（rollback なしで強制削除）
php artisan migrate:fresh
php artisan migrate:fresh --seed

# シーダーを実行（初期データ投入）
php artisan db:seed
php artisan db:seed --class=PostSeeder  # 特定のシーダーのみ実行

# DB に直接接続（対話型）
php artisan db
```

> `migrate:fresh` はすべてのデータが消えるため、**本番環境では絶対に使用しない**。

---

### 開発サーバー

```bash
# 開発用サーバーを起動（デフォルト: http://localhost:8000）
php artisan serve
php artisan serve --port=8080           # ポート指定
```

> Docker や Valet を使う場合は `artisan serve` は不要なことが多い。

---

### ルーティング確認

```bash
# 定義されているルートを一覧表示
php artisan route:list
php artisan route:list --path=api       # パスで絞り込み
php artisan route:list -v               # ミドルウェアも表示
```

---

### キャッシュ管理

パフォーマンス向上のためにキャッシュを生成し、開発中は変更が反映されない場合にクリアします。

```bash
# 設定ファイルをキャッシュ（本番環境向け）
php artisan config:cache
php artisan config:clear                # キャッシュをクリア

# ルートをキャッシュ（本番環境向け）
php artisan route:cache
php artisan route:clear

# ビュー（Blade テンプレート）をキャッシュ
php artisan view:cache
php artisan view:clear

# すべてのキャッシュを一括クリア
php artisan optimize:clear
```

> 本番環境では `php artisan optimize`（config + route + view のキャッシュを一括生成）を実行するのが一般的。

---

### キュー（非同期処理）

```bash
# キューワーカーを起動（バックグラウンドでジョブを処理）
php artisan queue:work
php artisan queue:work --queue=emails   # 特定のキューのみ処理

# 失敗したジョブを一覧表示
php artisan queue:failed

# 失敗したジョブを再実行
php artisan queue:retry all
php artisan queue:retry 5               # 特定のジョブIDを再実行
```

---

### 対話型シェル（Tinker）

```bash
# PHP の対話型 REPL を起動（モデル操作・動作確認に便利）
php artisan tinker
```

```php
// Tinker 上での操作例
>>> App\Models\Post::all();
>>> App\Models\User::factory()->create();
>>> DB::table('posts')->count();
```

---

### テスト

```bash
# テストを実行（Pest / PHPUnit）
php artisan test
php artisan test --filter=PostTest      # 特定のテストのみ実行
php artisan test --parallel             # 並列実行
```

---

## よく使うコマンドの組み合わせ

### 新機能を追加するとき

```bash
# 1. モデル + マイグレーション + コントローラ + ファクトリを一気に生成
php artisan make:model Post -mrc

# 2. マイグレーションを編集してからDBに反映
php artisan migrate

# 3. ルートを確認
php artisan route:list
```

### 開発中に環境をリセットしたいとき

```bash
# DB を作り直してシードデータを投入
php artisan migrate:fresh --seed
```

### 本番デプロイ時

```bash
# キャッシュを最適化
php artisan optimize

# キューワーカーを再起動（コード変更を反映）
php artisan queue:restart
```
