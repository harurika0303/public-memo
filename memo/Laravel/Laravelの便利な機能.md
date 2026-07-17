# Laravelの便利な機能

> 公式ドキュメント: https://laravel.com/docs

---

## 目次

1. [Artisan CLI](#1-artisan-cli)
2. [ルーティング](#2-ルーティング)
3. [Eloquent ORM](#3-eloquent-orm)
4. [マイグレーション](#4-マイグレーション)
5. [Bladeテンプレート](#5-bladeテンプレート)
6. [バリデーション](#6-バリデーション)
7. [認証・認可](#7-認証認可)
8. [セッション・フラッシュ](#8-セッションフラッシュ)
9. [CSRF保護](#9-csrf保護)
10. [コレクション](#10-コレクション)
11. [ヘルパー関数](#11-ヘルパー関数)
12. [サービスコンテナ・ファサード](#12-サービスコンテナファサード)
13. [メール・キュー・イベント](#13-メールキューイベント)
14. [その他の便利機能](#14-その他の便利機能)

---

## 1. Artisan CLI

> 公式: [Artisan Console](https://laravel.com/docs/artisan)

コマンド1つでファイルの雛形を自動生成してくれる。

```bash
# よく使うコマンド一覧
php artisan make:model Todo -mcr
# -m: マイグレーション同時作成
# -c: コントローラー同時作成
# -r: リソースコントローラー（index/create/store/...を自動定義）

php artisan make:controller TodoController   # コントローラー作成
php artisan make:model Todo                  # モデル作成
php artisan make:migration create_todos_table # マイグレーション作成
php artisan make:middleware CheckAge         # ミドルウェア作成
php artisan make:request StoreTodoRequest   # フォームリクエスト作成
php artisan make:seeder TodoSeeder          # シーダー作成
php artisan make:factory TodoFactory        # ファクトリー作成

php artisan migrate           # マイグレーション実行
php artisan migrate:rollback  # 直前のマイグレーションを元に戻す
php artisan migrate:fresh     # テーブルを全削除して再作成
php artisan migrate:fresh --seed  # テーブル再作成 + シーダー実行

php artisan route:list        # 定義済みルートを一覧表示
php artisan tinker            # 対話的にLaravelのコードを実行できるREPL
```

---

## 2. ルーティング

> 公式: [Routing](https://laravel.com/docs/routing)

### 名前付きルート

URLを直書きせず名前で参照できる。URLを変更しても1箇所修正するだけでよい。

```php
Route::get('/todos', [TodoController::class, 'index'])->name('todos.index');

// 使う側
route('todos.index')          // → "/todos"
redirect()->route('todos.index')
```

### ルートグループ・ミドルウェア

複数のルートに共通設定をまとめて適用できる。

```php
Route::middleware('auth')->prefix('admin')->group(function () {
    Route::get('/dashboard', ...);  // GET /admin/dashboard (要認証)
    Route::get('/users', ...);      // GET /admin/users     (要認証)
});
```

### ルートモデルバインディング

URLパラメーター `{todo}` をそのままEloquentモデルに自動変換してくれる。  
存在しないIDなら自動で404を返す。

```php
Route::delete('/todos/{todo}', [TodoController::class, 'destroy']);

// コントローラー側: $todo はすでにDBから取得済みのオブジェクト
public function destroy(Todo $todo)
{
    $todo->delete();
}
```

### リソースルート

CRUD操作の7つのルートを1行で定義できる。

```php
Route::resource('todos', TodoController::class);
// 上の1行で以下7ルートが自動生成される
// GET    /todos           index
// GET    /todos/create    create
// POST   /todos           store
// GET    /todos/{todo}    show
// GET    /todos/{todo}/edit  edit
// PUT    /todos/{todo}    update
// DELETE /todos/{todo}    destroy
```

---

## 3. Eloquent ORM

> 公式: [Eloquent: Getting Started](https://laravel.com/docs/eloquent)

### 基本的なCRUD

SQLを書かずにPHPのメソッドでDBを操作できる。

```php
// 全件取得
Todo::all();

// 条件付き取得
Todo::where('is_completed', false)->orderBy('order')->get();

// 1件取得（なければ404）
Todo::findOrFail($id);

// 作成
Todo::create(['title' => '牛乳を買う']);

// 更新
$todo->update(['is_completed' => true]);

// 削除
$todo->delete();
```

### リレーション

テーブル間の関連を定義するだけでJOINなしに関連データを取得できる。

```php
// User モデル
public function todos(): HasMany
{
    return $this->hasMany(Todo::class);  // 1対多
}

// 使う側
$user->todos;           // そのユーザーのTodo一覧（Collection）
$user->todos()->count() // Todoの件数
$user->todos()->create([...]) // そのユーザーのTodoを作成（user_idを自動セット）
```

| リレーション    | 意味                                 | メソッド         |
| --------------- | ------------------------------------ | ---------------- |
| `hasMany`       | 1対多（1ユーザーが多数のTodoを持つ） | `$user->todos`   |
| `belongsTo`     | 多対1（TodoはUserに属する）          | `$todo->user`    |
| `hasOne`        | 1対1                                 | `$user->profile` |
| `belongsToMany` | 多対多                               | `$post->tags`    |

### スコープ

よく使うクエリ条件を再利用可能な形でモデルに定義できる。

```php
// モデル側で定義
public function scopeCompleted(Builder $query): Builder
{
    return $query->where('is_completed', true);
}

// 使う側
Todo::completed()->get();  // WHERE is_completed = 1
```

### キャスト

DBのカラム値を自動でPHPの型に変換する。

```php
protected $casts = [
    'is_completed'     => 'boolean',  // 0/1 → true/false
    'password'         => 'hashed',   // 保存時に自動でbcryptハッシュ化
    'email_verified_at'=> 'datetime', // 文字列 → Carbonオブジェクト
];
```

### ソフトデリート

物理削除せず `deleted_at` に日時を記録するだけにする（論理削除）。復元もできる。

```php
use SoftDeletes;

$todo->delete();          // deleted_at に現在時刻を記録（物理削除しない）
$todo->restore();         // deleted_at を null に戻す
Todo::withTrashed()->get(); // 削除済みも含めて取得
```

---

## 4. マイグレーション

> 公式: [Database: Migrations](https://laravel.com/docs/migrations)

### テーブル操作

```php
// テーブル作成
Schema::create('todos', function (Blueprint $table) {
    $table->id();                              // BIGINT AUTO_INCREMENT PK
    $table->foreignId('user_id')->constrained()->cascadeOnDelete(); // 外部キー
    $table->string('title');                   // VARCHAR(255)
    $table->boolean('is_completed')->default(false);
    $table->integer('order')->default(0);
    $table->timestamps();                      // created_at, updated_at を自動追加
    $table->softDeletes();                     // deleted_at カラムを追加
});

// カラム追加
Schema::table('todos', function (Blueprint $table) {
    $table->boolean('is_completed')->default(false)->after('title');
});

// テーブル削除
Schema::dropIfExists('todos');
```

### ロールバック

`down()` に逆操作を書いておくことで `migrate:rollback` で元に戻せる。

```php
public function up(): void   { Schema::create('todos', ...); }
public function down(): void { Schema::dropIfExists('todos'); }
```

---

## 5. Bladeテンプレート

> 公式: [Blade Templates](https://laravel.com/docs/blade)

### テンプレート継承

共通レイアウトを継承して各ページを作る。

```blade
{{-- 親: layouts/app.blade.php --}}
<title>@yield('title', 'デフォルトタイトル')</title>
<body>@yield('content')</body>

{{-- 子: todos/index.blade.php --}}
@extends('layouts.app')
@section('title', 'Todo一覧')
@section('content')
    <h1>Todo一覧</h1>
@endsection
```

### よく使うディレクティブ

```blade
{{-- 変数出力（XSSエスケープあり） --}}
{{ $todo->title }}

{{-- 条件分岐 --}}
@if ($todos->isEmpty()) ... @else ... @endif
@auth ... @endauth          {{-- ログイン済みの場合のみ --}}
@guest ... @endguest        {{-- 未ログインの場合のみ --}}

{{-- ループ --}}
@foreach ($todos as $todo) ... @endforeach
@forelse ($todos as $todo) ... @empty <p>なし</p> @endforelse

{{-- フォーム関連 --}}
@csrf                       {{-- CSRFトークンのhidden inputを生成 --}}
@method('PATCH')            {{-- HTTPメソッドスプーフィング --}}
@checked($todo->is_completed)  {{-- checked属性の条件付き出力 --}}
@selected($val === $current)   {{-- selected属性の条件付き出力 --}}

{{-- 部分ビュー --}}
@include('todos.partials.alerts')
@include('todos.partials.item', ['todo' => $todo])

{{-- スタック（JSなどを子から親に積む） --}}
@push('scripts') <script>...</script> @endpush  {{-- 子で積む --}}
@stack('scripts')  {{-- 親で出力 --}}
```

---

## 6. バリデーション

> 公式: [Validation](https://laravel.com/docs/validation)

### コントローラーでの基本バリデーション

```php
$validated = $request->validate([
    'title'    => ['required', 'string', 'max:255'],
    'email'    => ['required', 'email', 'unique:users'],
    'password' => ['required', 'min:8', 'confirmed'],
    'ids.*'    => ['integer', 'exists:todos,id'],
]);
// 失敗 → 自動で前のページへリダイレクト + $errors にメッセージ格納
// 成功 → $validated にバリデーション済みの値が入る
```

### 主なバリデーションルール

| ルール            | 意味                               |
| ----------------- | ---------------------------------- |
| `required`        | 必須                               |
| `string`          | 文字列                             |
| `integer`         | 整数                               |
| `boolean`         | true/false                         |
| `email`           | メールアドレス形式                 |
| `max:255`         | 最大255文字                        |
| `min:8`           | 最小8文字                          |
| `unique:users`    | usersテーブルに重複なし            |
| `exists:todos,id` | todosテーブルのidに存在する        |
| `confirmed`       | `フィールド名_confirmation` と一致 |
| `array`           | 配列                               |
| `nullable`        | null許可                           |

### Form Request（バリデーションの分離）

バリデーションをコントローラーから切り出して専用クラスにまとめられる。

```bash
php artisan make:request StoreTodoRequest
```

```php
// app/Http/Requests/StoreTodoRequest.php
class StoreTodoRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return ['title' => ['required', 'string', 'max:255']];
    }
}

// コントローラー側はスッキリする
public function store(StoreTodoRequest $request)
{
    // $request は自動でバリデーション済み
}
```

---

## 7. 認証・認可

> 公式: [Authentication](https://laravel.com/docs/authentication) / [Authorization](https://laravel.com/docs/authorization)

### Auth ファサード

```php
Auth::user()           // ログイン中のUserモデルを取得
Auth::id()             // ログイン中のユーザーID
Auth::check()          // ログイン中かどうか（true/false）
Auth::login($user)     // 指定ユーザーをログイン状態にする
Auth::attempt($creds)  // メール・パスワードで認証試行
Auth::logout()         // ログアウト
```

### 認可（ポリシー）

誰が何を操作できるかのルールをモデルごとに定義できる。

```bash
php artisan make:policy TodoPolicy --model=Todo
```

```php
// app/Policies/TodoPolicy.php
public function update(User $user, Todo $todo): bool
{
    return $user->id === $todo->user_id;  // 自分のTodoのみ更新可
}

// コントローラー
$this->authorize('update', $todo);  // 失敗すると自動で403
```

### abort_unless（簡易認可）

ポリシーを使わず1行で認可チェックできる。

```php
abort_unless($todo->user_id === Auth::id(), 403);
```

---

## 8. セッション・フラッシュ

> 公式: [Session](https://laravel.com/docs/session) / [Flash Data](https://laravel.com/docs/session#flash-data)

```php
// セッションに保存（永続）
session(['key' => 'value']);
$request->session()->put('key', 'value');

// セッションから取得
session('key');

// フラッシュデータ（次のリクエストまで保持して自動削除）
redirect()->with('status', 'Todoを追加しました。');

// Bladeで取得
session('status')

// セッション再生成（セッション固定化攻撃対策）
$request->session()->regenerate();
```

---

## 9. CSRF保護

> 公式: [CSRF Protection](https://laravel.com/docs/csrf)

フォームやAjaxリクエストに自動でトークンを付与・検証する。悪意のあるサイトからの不正リクエストを防ぐ。

```blade
{{-- HTMLフォーム: @csrf でhidden inputを自動生成 --}}
<form method="POST">
    @csrf
</form>

{{-- Ajaxの場合: メタタグからトークンを取得してヘッダーに付与 --}}
<meta name="csrf-token" content="{{ csrf_token() }}">
```

```javascript
fetch('/todos/reorder', {
    method: 'PATCH',
    headers: { 'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').content }
});
```

---

## 10. コレクション

> 公式: [Collections](https://laravel.com/docs/collections)

Eloquentの取得結果（`get()` の戻り値）はCollectionで返る。  
配列より強力なメソッド群が使える。

```php
$todos = Todo::all(); // Collection

$todos->count();                         // 件数
$todos->isEmpty();                       // 空かどうか
$todos->first();                         // 先頭の要素
$todos->filter(fn($t) => $t->is_completed); // 完了済みのみ
$todos->map(fn($t) => $t->title);        // タイトルだけの配列に変換
$todos->pluck('title');                  // 特定カラムだけ取り出す
$todos->sortBy('order');                 // ソート
$todos->groupBy('user_id');              // グループ化
$todos->each(fn($t) => $t->update([...])); // 全件に処理
```

---

## 11. ヘルパー関数

> 公式: [Helpers](https://laravel.com/docs/helpers)

### よく使うグローバルヘルパー

```php
// URL関連
route('todos.index')          // 名前付きルートのURL生成
asset('css/app.css')          // publicディレクトリのURLを生成
url('/todos')                 // フルURLを生成

// リダイレクト
redirect()->route('todos.index')
redirect()->back()
redirect()->intended('/default')  // 元々行こうとしていたページへ

// レスポンス
response()->json(['ok' => true])
response()->view('todos.index', $data)

// 値関連
old('title')                  // 直前のリクエストの入力値
csrf_token()                  // CSRFトークン
app()->getLocale()            // 現在のロケール（ja など）
now()                         // 現在日時（Carbonオブジェクト）
```

---

## 12. サービスコンテナ・ファサード

> 公式: [Service Container](https://laravel.com/docs/container) / [Facades](https://laravel.com/docs/facades)

### ファサード

静的メソッドのような書き方でLaravelの機能にアクセスできる窓口。  
内部ではサービスコンテナからインスタンスを解決している。

| ファサード                  | 機能           |
| --------------------------- | -------------- |
| `Auth::user()`              | 認証           |
| `Route::get(...)`           | ルーティング   |
| `DB::table('todos')->get()` | クエリビルダー |
| `Schema::create(...)`       | スキーマ操作   |
| `Storage::put(...)`         | ファイル操作   |
| `Mail::to(...)->send(...)`  | メール送信     |
| `Cache::get('key')`         | キャッシュ     |
| `Log::info('message')`      | ログ出力       |
| `Str::upper('hello')`       | 文字列操作     |
| `Arr::get($array, 'key')`   | 配列操作       |

---

## 13. メール・キュー・イベント

### メール送信

> 公式: [Mail](https://laravel.com/docs/mail)

```php
Mail::to($user->email)->send(new WelcomeMail($user));
```

### キュー（バックグラウンド処理）

> 公式: [Queues](https://laravel.com/docs/queues)

時間のかかる処理（メール・画像処理など）をバックグラウンドで実行する。

```php
dispatch(new SendWelcomeEmailJob($user));  // ジョブをキューに投入
```

### イベント・リスナー

> 公式: [Events](https://laravel.com/docs/events)

「何かが起きた」ことを発火して、それを受け取って処理を実行する仕組み。

```php
event(new TodoCreated($todo));  // イベント発火

// リスナー側で受け取って処理
public function handle(TodoCreated $event): void
{
    // Todoが作成されたときの処理（通知など）
}
```

---

## 14. その他の便利機能

### ページネーション

> 公式: [Pagination](https://laravel.com/docs/pagination)

```php
// コントローラー
$todos = Todo::paginate(10);  // 10件ずつ

// Blade
{{ $todos->links() }}  // ページネーションリンクを自動表示
```

### ファイルストレージ

> 公式: [File Storage](https://laravel.com/docs/filesystem)

```php
Storage::put('path/to/file.txt', '内容');
Storage::get('path/to/file.txt');
Storage::delete('path/to/file.txt');
Storage::url('uploads/image.png');  // 公開URLを取得
```

### キャッシュ

> 公式: [Cache](https://laravel.com/docs/cache)

```php
Cache::put('todos', $todos, now()->addMinutes(10));  // 10分間キャッシュ
Cache::get('todos');         // キャッシュから取得
Cache::forget('todos');      // キャッシュ削除
Cache::remember('todos', 600, fn() => Todo::all()); // なければ実行してキャッシュ
```

### シーダー・ファクトリー（テストデータ）

> 公式: [Seeding](https://laravel.com/docs/seeding) / [Factories](https://laravel.com/docs/eloquent-factories)

```php
// ファクトリー: ランダムなテストデータを定義
class TodoFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title'        => $this->faker->sentence(),
            'is_completed' => $this->faker->boolean(),
        ];
    }
}

// 使い方
Todo::factory()->count(50)->create();  // テストデータを50件作成
```

### ログ

> 公式: [Logging](https://laravel.com/docs/logging)

```php
Log::info('Todoを追加しました', ['id' => $todo->id]);
Log::warning('注意メッセージ');
Log::error('エラーが発生しました');
// storage/logs/laravel.log に出力される
```

### .env と config

環境ごとに設定を切り替えられる。本番・開発・テストで値を変えられる。

```bash
# .env ファイル
DB_HOST=127.0.0.1
DB_DATABASE=todo_app
MAIL_MAILER=smtp
```

```php
// config/ から参照
config('database.default')

// .env から直接参照
env('DB_DATABASE', 'default_value')
```
