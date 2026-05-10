# SPA ハイブリッドとは

> 参考:
> - [Inertia.js — How It Works](https://inertiajs.com/docs/v3/core-concepts/how-it-works)
> - [Inertia.js — Who Is It For?](https://inertiajs.com/docs/v3/core-concepts/who-is-it-for)
> - [Inertia.js — The Protocol](https://inertiajs.com/docs/v3/core-concepts/the-protocol)

---

## 前提：3 つのアーキテクチャパターン

まず SPA ハイブリッドを理解するために、従来の 2 つのパターンとの違いを把握します。

### 1. MPA（Multi-Page Application）— 従来のサーバーサイドレンダリング

```
ブラウザがリンクをクリック
  → サーバーにリクエスト（HTML を要求）
  → サーバーが HTML を生成して返す（Blade, ERB など）
  → ブラウザがページ全体を再読み込み
  → 画面がチカッと切り替わる（フルページリロード）
```

**特徴：**
- ルーティング・認証・データ取得はすべてサーバー側で完結
- フロントエンドは HTML テンプレートを描画するだけ
- ページ遷移のたびに HTML を丸ごと返すため、体験がもっさりしやすい

---

### 2. SPA（Single-Page Application）— 純粋なクライアントサイドレンダリング

```
初回のみ HTML + JS をダウンロード
  → 以降のページ遷移は JS がクライアント側でコンポーネントを切り替え
  → データはバックエンド API（REST / GraphQL）から JSON で取得
  → URL が変わっても画面は JS が書き換えるだけ（フルリロードなし）
```

**特徴：**
- ページ遷移が瞬時でスムーズ（シームレスな体験）
- バックエンドは API を提供するだけ（フロントエンドと完全分離）
- ただしバックエンドと別に API 設計・認証・状態管理が必要になり、構成が複雑になる

---

### 3. SPA ハイブリッド — Inertia.js が実現するアプローチ

**「サーバーサイドのシンプルさ」 ＋ 「SPA のスムーズな体験」を両立させる**

```
ブラウザがリンクをクリック
  → Inertia が XHR リクエストを送信（フルリロードではない）
  → サーバーは HTML ではなく JSON（コンポーネント名 ＋ データ）を返す
  → クライアントが React/Vue/Svelte コンポーネントを差し替える
  → 画面の切り替えは JS が行う（SPA と同じ体験）
```

API を別途構築する必要がなく、Laravel のルーター・コントローラ・認証をそのまま使えます。

---

## Inertia.js の仕組み

SPA ハイブリッドを実現するのが **Inertia.js** です。

### 初回リクエスト（フルページ）

初回アクセス時は通常の HTML レスポンスを返します。この HTML には React/Vue/Svelte アプリのマウントポイントと、初期データ（JSON）が埋め込まれています。

```
GET /users
Accept: text/html

↓ レスポンス（HTML）

<html>
  <head>
    <script src="/js/app.js" defer></script>
  </head>
  <body>
    <script type="application/json" data-page="app">
      {"component":"Users","props":{"users":[...]},"url":"/users"}
    </script>
    <div id="app"></div>
  </body>
</html>
```

---

### 2 回目以降のリクエスト（Inertia リクエスト）

`<Link>` コンポーネントをクリックすると、Inertia がリンクのクリックを横取りして XHR リクエストを送ります。サーバーは `X-Inertia: true` ヘッダーを検出し、HTML ではなく **JSON** を返します。

```
GET /users/1
X-Inertia: true
X-Inertia-Version: abc123

↓ レスポンス（JSON）

{
  "component": "UserDetail",
  "props": { "user": { "id": 1, "name": "Alice" } },
  "url": "/users/1"
}
```

Inertia はこの JSON を受け取り、画面のコンポーネントを差し替えます。ブラウザの URL も更新されます（History API）。

---

### リクエストライフサイクルの比較

```
【通常の SPA（API 分離）】
ブラウザ → フロントエンド（React/Next.js）→ API リクエスト → Laravel API → JSON → 画面更新

【Inertia.js（SPA ハイブリッド）】
ブラウザ → Inertia（XHR）→ Laravel コントローラ → Inertia::render() → JSON → 画面更新
```

---

## コード例（Laravel ＋ Inertia.js ＋ Vue）

### Laravel 側（コントローラ）

```php
// app/Http/Controllers/UserController.php
use Inertia\Inertia;

class UserController extends Controller
{
    public function index()
    {
        $users = User::active()->orderBy('name')->get(['id', 'name', 'email']);

        // Blade テンプレートの代わりに Vue コンポーネントを指定
        return Inertia::render('Users', [
            'users' => $users,
        ]);
    }
}
```

### Vue 側（ページコンポーネント）

```vue
<!-- resources/js/Pages/Users.vue -->
<script setup lang="ts">
import { Link } from '@inertiajs/vue3'
defineProps<{ users: User[] }>()
</script>

<template>
  <div v-for="user in users" :key="user.id">
    <!-- Link コンポーネントがクリックを横取りし XHR で遷移 -->
    <Link :href="`/users/${user.id}`">
      {{ user.name }}
    </Link>
    <p>{{ user.email }}</p>
  </div>
</template>
```

- `Inertia::render('Users', [...])` は Blade の `view('users', [...])` と同じ感覚で書ける
- `<Link>` は `<a>` タグの代替。クリック時にフルリロードせず XHR で遷移する

---

## 3 パターンの比較まとめ

| 項目           | MPA（Blade など）  | SPA（React + API）           | SPA ハイブリッド（Inertia.js） |
| -------------- | ------------------ | ---------------------------- | ------------------------------ |
| ページ遷移     | フルリロード       | JS で差し替え（高速）        | JS で差し替え（高速）          |
| バックエンド   | テンプレートを返す | JSON API を提供              | JSON + コンポーネント名を返す  |
| API 設計       | 不要               | 必要（REST / GraphQL）       | **不要**                       |
| フロントエンド | HTML テンプレート  | React / Vue / Svelte         | React / Vue / Svelte           |
| 認証           | サーバー側で完結   | API 用トークン認証が必要     | **サーバー側で完結**           |
| 状態管理       | 不要               | Pinia / Zustand など必要     | 最小限でよい                   |
| SEO            | 良い               | 工夫が必要（SSR など）       | SSR オプションあり             |
| 構成の複雑さ   | シンプル           | 複雑（フロント・バック分離） | シンプル（モノレポで完結）     |

---

## 「SPA ハイブリッド」という名称について

公式ドキュメントでは **"modern monolith"（モダンモノリス）** とも呼んでいます。

- **モノリス**: バックエンドとフロントエンドが 1 つのコードベースに同居（API サーバーと別フロントのリポジトリに分かれない）
- **モダン**: ビューは React/Vue/Svelte で構築し、SPA レベルの UX を実現

「ハイブリッド」とは「サーバーサイドのシンプルさ」と「SPA のスムーズな体験」を組み合わせていることを指しています。

---

## Inertia.js が向いているケース / 向いていないケース

| 向いている                                        | 向いていない                                                               |
| ------------------------------------------------- | -------------------------------------------------------------------------- |
| Laravel や Rails を使っている既存チーム           | フロントエンドを完全に分離したい場合（別チーム・別リポジトリ）             |
| SPA の UX を出したいが API 設計のコストを避けたい | モバイルアプリとも同じ API を共有したい場合（→ Sanctum + REST の方が向く） |
| フルスタック開発者がひとりで開発する場合          | SEO を最優先し SSR を徹底したい場合（→ Next.js などが有利）                |
| 管理画面・業務システムなど社内ツール              |                                                                            |
