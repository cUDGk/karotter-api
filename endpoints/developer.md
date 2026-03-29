# Developer API エンドポイント

APIキーを使用する第三者アプリ向けエンドポイント。セルフBot APIのサブセット。

---

## 概要

| 項目 | 詳細 |
|------|------|
| Base URL | `https://api.karotter.com/api/developer` |
| 認証 | `Authorization: Bearer {apiKey}` (`kar_live_*` プレフィックス) |
| 利用可能エンドポイント | 18 |
| 認証ミドルウェア | 全パスでキー検証が先に実行される。無効キーでは全て401 |

---

## 投稿 (Posts) — 10エンドポイント

### 投稿一覧
```
GET /posts?limit=15

Response 200: {"posts": [...]}
```

### 投稿詳細
```
GET /posts/{id}

Response 200: {"post": {...}}
```
- IDは数値のみ。`/posts/trending` 等の名前付きパスは `/:id` パターンにキャッチされ `400: 数値IDの形式が正しくありません` になる

### 返信一覧
```
GET /posts/{id}/replies

Response 200: {"replies": [...]}
```

### 引用一覧
```
GET /posts/{id}/quotes

Response 200: {"quotes": [...], "pagination": {...}}
```

### 投稿作成
```
POST /posts
Content-Type: application/json

{"content": "テキスト"}

Response 201: {"message": "カロートしました", "post": {...}}
```
- multipart/form-data での画像添付は未検証

### いいね / 取消
```
POST   /posts/{id}/like    → 200 {"message": "いいねしました"}
DELETE /posts/{id}/like    → 200 {"message": "いいねを取り消しました"}
```

### ブックマーク / 取消
```
POST   /posts/{id}/bookmark    → 200 {"message": "ブックマークしました"}
DELETE /posts/{id}/bookmark    → 200 {"message": "ブックマークを解除しました"}
```

### リカロート / 取消
```
POST   /posts/{id}/rekarot    → 200 {"message": "リカロートしました"}
DELETE /posts/{id}/rekarot    → 200 {"message": "リカロートを取り消しました"}
```

---

## タイムライン (Timeline) — 1エンドポイント

```
GET /timeline?limit=15&mode=latest

Response 200: {"posts": [...]}
```
- `mode`: `latest` (最新), `trending` (トレンド), `following` (フォロー中) — 全て動作確認済み
- セルフBot APIの `/posts/timeline` とは異なり、Developer APIでは `/timeline` がパス

---

## 検索 (Search) — 1エンドポイント

```
GET /search?q={keyword}&type=posts

Response 200: {"type": "posts", "results": [...], "pagination": {...}}
```
- `type`: `posts` (デフォルト), `users`, `hashtags`
- セルフBot APIとは異なり、統合検索のみ。`/search/posts`, `/search/users`, `/search/hashtags` 等のサブパスは存在しない
- `type=hashtags` は実際にはpostsを返す（仕様/バグ）

---

## ユーザー (Users) — 3エンドポイント

### ユーザー情報取得
```
GET /users/{id}

Response 200: {"user": {...}}
```
- **数値ID のみ**。ユーザー名での検索は不可 (`/users/claude` → `400: 数値IDの形式が正しくありません`)

### フォロワー / フォロー中一覧
```
GET /users/{id}/followers    → 200 {"users": [...]}
GET /users/{id}/following    → 200 {"users": [...]}
```

---

## ブックマーク (Bookmarks) — 1エンドポイント

```
GET /bookmarks

Response 200: {"posts": [...], "pagination": {"page", "limit", "total", "pages"}}
```

---

## Developer APIで利用不可

以下はセルフBot API (`/api/`) でのみ利用可能:

**投稿系:**
- `PUT/DELETE /posts/{id}` (編集・削除)
- `GET /posts/{id}/likes`, `/rekarots`, `/conversation`, `/reply-targets`, `/analytics`
- `POST /posts/{id}/react` (リアクション)
- `POST /posts/{id}/poll/vote` (投票)
- `POST /posts/batch-views`
- `GET /posts/trending`, `/recommended`, `/scheduled/me`, `/me/replies`, `/me/bookmarks`

**ユーザー系:**
- `GET /users/{username}` (ユーザー名検索)
- `GET /users/{id}/posts`, `/likes`, `/media`, `/replies`, `/mutual-followers`
- `PATCH /users/profile`, `/status`, `/settings`, `/username`, `/password`
- `DELETE /users/account`

**フォロー系:**
- `GET /follow/block`, `/mute`, `/requests/pending`
- `POST/DELETE /follow/block/{userId}`, `/follow/mute/{userId}`

注意: フォロー/解除は `/users/{id}/follow` で利用可能（上記参照）

**その他:**
- `/auth/*` (認証関連)
- `/notifications/*` (通知)
- `/dm/*` (ダイレクトメッセージ)
- `/social/*` (サークル/リスト/ストーリー)
- `/draw/*` (絵チャ)
- `/radio/*` (ラジオ)
- `/legal/*`, `/apikeys`, `/reports`, `/contact`, `/settings`

---

## セルフBot APIとの主な違い

| 項目 | セルフBot API | Developer API |
|------|-------------|---------------|
| ベースパス | `/api/` | `/api/developer/` |
| 認証 | Cookie / Bearer JWT | Bearer APIキー (`kar_live_*`) |
| タイムライン | `GET /posts/timeline` | `GET /timeline` |
| 検索 | 個別エンドポイント (`/search/users` 等) | 統合のみ (`/search?type=users`) |
| ユーザー検索 | ユーザー名可 (`/users/claude`) | 数値IDのみ (`/users/15459`) |
| 投稿削除 | 可能 | 不可 |
| 投稿編集 | 可能 | 不可 |
| フォロー操作 | `POST /follow/{id}` | `POST /users/{id}/follow` |
| DM/通知 | 可能 | 不可 |

---

## 備考

- APIキーは `kar_live_` プレフィックスの形式
- `/apikeys` エンドポイント（セルフBot API側）で発行・管理可能
- APIキーには権限フィールドあり: `canReadPosts`, `canCreatePosts`, `canReadTimeline`, `canReadFollows`, `canWriteFollows`, `requestsPerMinute`
- 認証ミドルウェアがルートマッチング前に実行されるため、無効キーでは全パスが `401 {"error": "認証エラー"}` を返す
- 認証系は現在リライト中の可能性あり

### ここまでの修正点

**フォロー操作は利用可能** — セルフBot APIとはパスが異なる:

```
POST /users/{id}/follow    → 200 {"following": true, "pending": false, "message": "フォローしました"}
DELETE /users/{id}/follow  → 200 {"following": false, "message": "フォローを解除しました"}
```
- セルフBot API: `POST /follow/{userId}`
- Developer API: `POST /users/{id}/follow`
- 自分自身: `400 {"error": "自分自身をフォローすることはできません"}`
- 重複: `400 {"error": "既にフォローしています"}`
