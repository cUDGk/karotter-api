# Karotter API Reference

Karotter (カロッター) の非公式APIリファレンスです。
フロントエンドのJSバンドル解析、実際のAPIテスト、セキュリティ診断レポート、および公式ドキュメントに基づいて作成しています。

---

## Base URL

| 用途 | Base URL |
|------|----------|
| セルフBot (自アカウント操作) | `https://api.karotter.com/api/` |
| 開発者向け (第三者アプリ) | `https://api.karotter.com/api/developer` |

> `https://karotter.com/api` でもアクセス可能（Cloudflare経由）

---

## 共通ヘッダー

全リクエストに以下のヘッダーを付与する。

| ヘッダー | 値 | 必須 | 備考 |
|---------|-----|------|------|
| `X-Client-Type` | `web` | 推奨 | `web` 以外でも動作するがアカウント凍結リスクあり |
| `X-Device-Id` | UUID v4 | 推奨 | セッション識別用、固定値でOK |
| `X-CSRF-Token` | CSRFトークン | 書き込み系 | POST/PUT/DELETE で必要 |

### 認証方式

**方式1: APIキー認証** (開発者向け)
```
Authorization: Bearer {apiKey}
```
- `/apikeys` エンドポイントで発行可能

**方式2: Cookie認証** (セルフBot)
```
Cookie: {ログイン時に返却されるCookie}
```
- ログインレスポンスの `Set-Cookie` をそのまま使用

---

## 認証 (Auth)

### ログイン
```
POST /auth/login
Content-Type: application/json

{
  "identifier": "username",
  "password": "password",
  "gender": "OTHER"
}

Response 200:
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ...",
  "user": { ... }
}
```
- `accessToken`: JWT (HS256), 30分有効
- `refreshToken`: JWT (HS256), 30日有効
- レスポンスの `Set-Cookie` にCookieも含まれる

### CSRFトークン取得
```
GET /auth/csrf-token

Response 200:
{
  "csrfToken": "f7eb0b1a-..."
}
```
- 書き込み系リクエスト (POST/PUT/DELETE) に `X-CSRF-Token: {csrfToken}` が必要

### トークンリフレッシュ
```
POST /auth/refresh-token
Cookie: karotter_rt={refreshToken}
```

### ログアウト
```
POST /auth/logout
```

### 自分の情報取得
```
GET /auth/me

Response 200:
{
  "id": 15459,
  "username": "claude",
  "displayName": "claude",
  "avatarUrl": "/uploads/avatars/...",
  "bio": "...",
  "followersCount": 167,
  "followingCount": 172,
  "postsCount": 1131,
  ...
}
```

### アカウント登録
```
POST /auth/register
Content-Type: application/json

{
  "username": "...",
  "email": "...",
  "password": "...",
  "gender": "OTHER"
}
```
- パスワード: 8文字以上

### パスワードリセット
```
POST /auth/forgot-password
POST /auth/reset-password
```

### メール認証
```
POST /auth/verify-email
POST /auth/me/email/resend
```

### セッション管理
```
GET  /auth/sessions                  → 全セッション一覧
DELETE /auth/sessions/:id            → 特定セッション削除
DELETE /auth/sessions/others         → 他の全セッションを削除
POST /auth/switch-session            → セッション切り替え
POST /auth/session-unread-snapshots  → 未読スナップショット
```

---

## 投稿 (Posts)

### カロート投稿
```
POST /posts
Content-Type: multipart/form-data
Authorization: Bearer {token}
X-CSRF-Token: {csrf}

Fields:
  content          : string (テキスト)
  media            : File[] (画像/動画ファイル、同名で複数append可)
  mediaAlts        : string (JSON配列 e.g. ["alt text", ""])
  mediaSpoilerFlags: string (JSON配列 e.g. [false, false])
  mediaR18Flags    : string (JSON配列 e.g. [false, false])
  isAiGenerated    : string ("true" / "false")
  isPromotional    : string ("true" / "false")
  isR18            : string ("true" / "false")
  hideFromMinors   : string ("true" / "false")
  visibility       : string ("PUBLIC" / "FOLLOWERS" / "CIRCLE" / "PRIVATE")
  replyRestriction : string ("EVERYONE" / "FOLLOWERS" / "CIRCLE" / "MENTIONS")
  parentId         : string (返信先の投稿ID)
  quotedPostId     : string (引用元の投稿ID)
  pollOptions      : string (JSON配列 e.g. ["選択肢1", "選択肢2"])
  pollDurationHours: string (投票期間)
  scheduledFor     : string (ISO 8601, 予約投稿日時)
  viewerCircleId   : string (サークル限定時)
  replyCircleId    : string (返信制限サークル)
  excludedMentions : string (JSON配列, メンション除外ユーザーID)

Response 201:
{
  "message": "カロートしました",
  "post": { ... }
}
```

**制限事項**:
- 画像と動画の同時投稿不可 (`400: 画像と動画は同時に送信できません`)
- フィールド名は `media` (投稿) / `attachments` (DM) — 間違えると500エラー

### 投稿取得
```
GET /posts/{id}
Authorization: Bearer {token}

Response 200:
{
  "post": {
    "id": 94803,
    "content": "テキスト",
    "authorId": 15459,
    "parentId": null,
    "quotedPostId": null,
    "mediaUrls": ["/uploads/posts/uuid.png"],
    "mediaTypes": ["image"],
    "mediaAlts": [""],
    "mediaSpoilerFlags": [false],
    "mediaR18Flags": [false],
    "likesCount": 6,
    "rekarotsCount": 0,
    "repliesCount": 1,
    "reactionsCount": 2,
    "liked": false,
    "rekaroted": false,
    "bookmarked": false,
    "author": { ... },
    "poll": null | { ... },
    "reactions": [...],
    "reactionSummary": [...],
    "createdAt": "2026-03-27T...",
    ...
  }
}
```

### 投稿編集
```
PUT /posts/{id}
Content-Type: multipart/form-data
```

### 投稿削除
```
DELETE /posts/{id}
Authorization: Bearer {token}
X-CSRF-Token: {csrf}

Response 200:
{
  "message": "カロートを削除しました"
}
```
- 他人の投稿: `403: このカロートを削除する権限がありません`
- 削除後のGET: `404: カロートが見つかりません`

### タイムライン
```
GET /posts/timeline?page=1&limit=15&mode=latest
Authorization: Bearer {token}

Response 200:
{
  "posts": [...],
  "pagination": { "page": 1, "limit": 15, "mode": "latest" }
}
```
- `limit` 最大値: 100 (サーバー側でキャップ)
- `mode`: `latest`

### トレンド
```
GET /posts/trending
```
- 認証不要

### おすすめ投稿
```
GET /posts/recommended
```

### 予約投稿一覧
```
GET /posts/scheduled/me
```

### 閲覧数バッチ送信
```
POST /posts/batch-views
```

### 返信先ユーザー取得
```
GET /posts/{id}/reply-targets
```

---

## いいね (Like)

### いいねする
```
POST /posts/{id}/like
Authorization: Bearer {token}
X-CSRF-Token: {csrf}

Response 200:
{
  "message": "いいねしました"
}
```

### いいね取り消し
```
DELETE /posts/{id}/like

Response 200:
{
  "message": "いいねを取り消しました"
}
```
- エンドポイントは `/like` (単数形)。`/likes` は404

---

## リアクション (Reaction)

### リアクション追加
```
POST /posts/{id}/react
Content-Type: application/json

{
  "emoji": "👍"
}

Response 200:
{
  "message": "リアクションしました"
}
```
- 絵文字は32文字以内 (任意文字列可)
- 同じ絵文字で2回目: `400: 既にこの絵文字でリアクションしています`

### リアクション取り消し
```
DELETE /posts/{id}/react/{emoji}

Response 200:
{
  "message": "リアクションを取り消しました"
}
```
- 絵文字はURLエンコードして渡す (e.g. `/react/%F0%9F%91%8D`)

---

## 投票 (Poll)

### 投票する
```
POST /posts/{id}/poll/vote
Content-Type: application/json

{
  "optionId": 123
}
```
- トグル式: 再度POSTで投票取り消し
- 投票データは `GET /posts/{id}` の `post.poll` に含まれる

---

## フォロー

### フォローする
```
POST /follow/{userId}
Authorization: Bearer {token}
X-CSRF-Token: {csrf}
```

### フォロワー一覧
```
GET /users/{userId}/followers?limit=100
```

### フォロー中一覧
```
GET /users/{userId}/following?limit=1000
```

### フォロー解除
```
DELETE /follow/{userId}
```

### フォローリクエスト (保留中)
```
GET /follow/requests/pending
```

### ブロック一覧
```
GET /follow/block
```

### ミュート一覧
```
GET /follow/mute
```

---

## ユーザー

### ユーザー情報取得
```
GET /users/{username}
Authorization: Bearer {token}

Response 200:
{
  "user": {
    "id": 15459,
    "username": "claude",
    "displayName": "claude",
    "avatarUrl": "...",
    "headerUrl": "...",
    "bio": "...",
    "isBotAccount": true,
    "isParodyAccount": false,
    "hideProfileFromMinors": true,
    "isPrivate": false,
    "onlineStatus": "ONLINE",
    "followersCount": 167,
    "followingCount": 172,
    "postsCount": 1131,
    ...
  },
  "isFollowing": false,
  "isFollowedBy": false,
  "isBlocked": false,
  ...
}
```

### ユーザーの投稿一覧
```
GET /users/{userId}/posts
```

### おすすめユーザー
```
GET /users/recommended?limit=3
```

### ユーザー名クォータ確認
```
GET /users/username/quota
```

### プロフィール更新
```
PATCH /users/profile
```

### ステータス更新
```
PUT /users/status
```

### アバター更新
```
POST /profile/avatar
Content-Type: multipart/form-data
```

---

## 通知 (Notifications)

### 通知一覧
```
GET /notifications?limit=30
Authorization: Bearer {token}

Response 200:
{
  "notifications": [
    {
      "id": "...",
      "type": "REPLY",
      "actor": { ... },
      "actors": [{ ... }],
      "post": { ... },
      "posts": [{ ... }],
      "createdAt": "..."
    },
    ...
  ]
}
```
- `type`: `REPLY`, `MENTION`, `FOLLOW`, `LIKE`, `REKAROT`

### 未読通知数
```
GET /notifications/unread/count
```

### グループ化された投稿通知
```
GET /notifications/grouped-posts
```

### プッシュ通知登録
```
POST /notifications/push/register
POST /notifications/push/unregister
```

---

## リカロート (Rekarot)

### リカロートする
```
POST /posts/{id}/rekarot
Authorization: Bearer {token}
X-CSRF-Token: {csrf}
```

---

## ブックマーク

### ブックマーク一覧
```
GET /posts/me/bookmarks
Authorization: Bearer {token}
```

### ブックマーク追加
```
POST /posts/{id}/bookmark
Authorization: Bearer {token}
X-CSRF-Token: {csrf}
```

---

## DM (ダイレクトメッセージ)

### DMグループ一覧
```
GET /dm/groups
Authorization: Bearer {token}

Response 200:
{
  "groups": [
    {
      "id": 487,
      "members": [...],
      "lastMessage": { ... },
      "messages": [...],
      ...
    }
  ]
}
```

### メッセージ取得
```
GET /dm/groups/{groupId}/messages?page=1&limit=50
Authorization: Bearer {token}

Response 200:
{
  "messages": [
    {
      "id": 4125,
      "groupId": 487,
      "senderId": 15459,
      "content": "テキスト",
      "replyToId": null,
      "attachmentUrls": ["/uploads/dm/uuid.png"],
      "attachmentTypes": ["image/png"],
      "attachmentAlts": [""],
      "attachmentSpoilerFlags": [false],
      "attachmentR18Flags": [false],
      "isDeleted": false,
      "createdAt": "...",
      "sender": { ... },
      "reactions": [...]
    }
  ],
  "pagination": { ... }
}
```

### メッセージ送信
```
POST /dm/groups/{groupId}/messages
Content-Type: multipart/form-data
Authorization: Bearer {token}
X-CSRF-Token: {csrf}

Fields:
  content               : string
  attachments           : File[] (画像ファイル、同名で複数append可)
  attachmentAlts        : string (JSON配列)
  attachmentSpoilerFlags: string (JSON配列)
  attachmentR18Flags    : string (JSON配列)
  replyToId             : string (返信先メッセージID)
  pollOptions           : string (JSON配列)
  pollDurationHours     : string

Response 201:
{
  "message": {
    "id": 4125,
    "groupId": 487,
    ...
  }
}
```
- フィールド名は `attachments` (投稿の `media` とは異なる)

### 既読マーク
```
POST /dm/groups/{groupId}/read
```

### 通話
```
POST /dm/groups/{groupId}/call/start
POST /dm/groups/{groupId}/call/join
POST /dm/groups/{groupId}/call/leave
```

---

## 検索 (Search)

### 統合検索
```
GET /search?q={keyword}
```

### 投稿検索
```
GET /search/posts?q={keyword}
GET /posts/search?q={keyword}
```

### ユーザー検索
```
GET /search/users?q={keyword}
```

### ハッシュタグ検索
```
GET /search/hashtags?q={keyword}
```

### トレンドトピック
```
GET /search/trending/topics?limit=5
```

### ディスカバー (最新投稿)
```
GET /search/discover/latest?limit=12&cursor={lastPostId}
```
- `limit`: 取得件数 (デフォルト12)
- `cursor`: 取得開始位置。指定したIDより前の投稿を取得
- 初回は `cursor` なしで最新を取得、以降は最後の投稿IDを `cursor` にセットして無限スクロール

---

## Draw-chat (絵チャ)

### ルーム一覧
```
GET /draw/rooms
```

### ルーム作成
```
POST /draw/rooms
Content-Type: application/json

{
  "name": "ルーム名"
}
```

### ルーム詳細 (レイヤー含む)
```
GET /draw/rooms/{roomId}
```

### レイヤー更新
```
PUT /draw/rooms/{roomId}/layers
Content-Type: application/json

{
  "layers": [
    {
      "id": "layerId",
      "name": "レイヤー名",
      "order": 0,
      "visible": true,
      "opacity": 1,
      "dataUrl": "data:image/png;base64,..."
    }
  ]
}
```
- キャンバス: 2560x2560px RGBA PNG
- 合計ペイロード上限: 約5MB
- 新レイヤー追加時は既存レイヤーも含めて一括送信 (しないと既存レイヤーが消える)

### ルーム参加
```
POST /draw/rooms/{roomId}/join
```

### ルーム削除
```
DELETE /draw/rooms/{roomId}
```
- オーナーのみ

---

## ラジオ (Radio)

### アクティブなラジオ一覧
```
GET /radio/active
```

### ラジオ詳細
```
GET /radio/{id}
```

### ラジオ参加 / 退出 / 終了
```
POST /radio/{id}/join
POST /radio/{id}/leave
POST /radio/{id}/end
```

---

## ソーシャル

### サークル一覧
```
GET /social/circles
```

### リスト一覧
```
GET /social/lists
```

### ストーリー
```
GET /social/stories
```

### リンクプレビュー
```
GET /social/link-preview?url={url}
```
- URLのOGP情報を取得してプレビューを生成

---

## レポート / お問い合わせ

### 通報
```
POST /reports
Content-Type: application/json
```

### お問い合わせ
```
POST /contact
Content-Type: application/json
```

---

## APIキー

```
GET  /apikeys    → APIキー一覧
POST /apikeys    → APIキー作成
```

---

## Developer API

APIキーを使用する第三者アプリ向けエンドポイント。セルフBot APIのサブセット。

**Base URL**: `https://api.karotter.com/api/developer`

**認証**: `Authorization: Bearer {apiKey}` (APIキーは `kar_live_` プレフィックス)

### 利用可能エンドポイント

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/posts` | 自分の投稿一覧 |
| GET | `/posts/{id}` | 投稿詳細 |
| GET | `/posts/{id}/replies` | 返信一覧 |
| POST | `/posts` | 投稿作成 |
| GET | `/timeline` | タイムライン |
| GET | `/search?q={keyword}` | 投稿検索 |
| GET | `/bookmarks` | ブックマーク一覧 |
| POST | `/posts/{id}/like` | いいね |
| DELETE | `/posts/{id}/like` | いいね取消 |
| POST | `/posts/{id}/bookmark` | ブックマーク追加 |
| POST | `/posts/{id}/rekarot` | リカロート |

### Developer APIで利用不可

以下はセルフBot API (`/api/`) でのみ利用可能:

- `DELETE /posts/{id}` (投稿削除)
- `/auth/*` (認証関連)
- `/notifications/*` (通知)
- `/dm/*` (ダイレクトメッセージ)
- `/users/*` (ユーザー情報)
- `/follow/*` (フォロー操作)
- `/draw/*` (絵チャ)
- `/radio/*` (ラジオ)

---

## Socket.IO (リアルタイム通信)

**接続先**: `wss://karotter.com/socket.io`

### DMイベント
| イベント | 方向 | データ |
|---------|------|-------|
| `joinDM` | Client → Server | `{ groupId }` |
| `leaveDM` | Client → Server | `{ groupId }` |
| `markAsRead` | Client → Server | `{ groupId, messageId }` |
| `startTyping` | Client → Server | `{ groupId }` |
| `stopTyping` | Client → Server | `{ groupId }` |

### 通話イベント
| イベント | 方向 | データ |
|---------|------|-------|
| `voice:offer` | Client → Server | WebRTC SDP |
| `voice:answer` | Client → Server | WebRTC SDP |
| `voice:ice-candidate` | Client → Server | ICE candidate |
| `voice:hangup` | Client → Server | - |

### 絵チャイベント
| イベント | 方向 | データ |
|---------|------|-------|
| `draw:join` | Client → Server | `{ roomId }` |
| `draw:room-state` | Server → Client | ルーム状態 |

---

## メディア

### メディアURLの構造
```
画像: /uploads/posts/{uuid}.{ext}
DM添付: /uploads/dm/{uuid}.{ext}
アバター: /uploads/avatars/avatar_{userId}_{timestamp}.webp
ヘッダー: /uploads/headers/header_{userId}_{timestamp}.webp

フルURL: https://karotter.com{path}
```

### メディアフィールド (投稿)
```json
{
  "mediaUrls": ["/uploads/posts/uuid.png"],
  "mediaTypes": ["image"],
  "mediaAlts": [""],
  "mediaSpoilerFlags": [false],
  "mediaR18Flags": [false]
}
```
- `mediaTypes`: `"image"` / `"video"`

### メディアフィールド (DM)
```json
{
  "attachmentUrls": ["/uploads/dm/uuid.png"],
  "attachmentTypes": ["image/png"],
  "attachmentAlts": [""],
  "attachmentSpoilerFlags": [false],
  "attachmentR18Flags": [false]
}
```
- フィールド名が投稿とは異なる (`media*` vs `attachment*`)
- `attachmentTypes` は MIME type (`image/png` 等)

---

## クエリパラメータ

多くのエンドポイントで以下のパラメータが使用可能。

### ページネーション

**Offset方式** (従来):
```
?page=1&limit=15
```

**Cursor方式** (推奨、無限スクロール用):
```
?limit=12&cursor={lastId}
```
1. 初回は `cursor` なしでリクエスト → 最新N件を取得
2. レスポンスの最後の投稿IDを `cursor` に指定 → それ以前のデータを取得
3. データの重複なく過去を遡れる

---

## 投稿レスポンス構造

```json
{
  "id": 12345,
  "content": "投稿テキスト @mention #hashtag",
  "createdAt": "2026-03-28T06:50:55.026Z",
  "authorId": 15459,
  "parentId": null,
  "quotedPostId": null,
  "mediaUrls": [],
  "mediaTypes": [],
  "mediaAlts": [],
  "mediaSpoilerFlags": [],
  "mediaR18Flags": [],
  "isAiGenerated": false,
  "isPromotional": false,
  "isR18": false,
  "hideFromMinors": false,
  "visibility": "PUBLIC",
  "replyRestriction": "EVERYONE",
  "likesCount": 6,
  "rekarotsCount": 0,
  "repliesCount": 1,
  "reactionsCount": 2,
  "viewsCount": 100,
  "liked": false,
  "rekaroted": false,
  "bookmarked": false,
  "author": {
    "id": 15459,
    "username": "claude",
    "displayName": "claude",
    "avatarUrl": "/uploads/avatars/...",
    "isBotAccount": true,
    "isParodyAccount": false,
    "officialMark": "NONE"
  },
  "poll": null,
  "reactions": [{"emoji": "🥕", "userId": 15459}],
  "reactionSummary": [{"emoji": "🥕", "count": 1, "reacted": true}],
  "mentions": [],
  "hashtags": [],
  "editedAt": null
}
```

### 主要フィールド

| フィールド | 説明 |
|-----------|------|
| `id` | 投稿ID |
| `content` | 本文 (メンション・ハッシュタグ含む) |
| `createdAt` | 投稿日時 (ISO 8601) |
| `parentId` | 返信先投稿ID (スレッド構造用、ルート投稿は`null`) |
| `quotedPostId` | 引用元投稿ID |
| `mediaUrls` | メディアの相対パス配列 |
| `mediaTypes` | メディア種別 (`"image"` / `"video"`) |
| `liked` / `rekaroted` / `bookmarked` | 自分がアクションしたか |
| `author` | 投稿者情報 (埋め込み) |
| `poll` | 投票データ (`null` or オブジェクト) |
| `reactions` | リアクション配列 |
| `visibility` | `PUBLIC` / `FOLLOWERS` / `CIRCLE` / `PRIVATE` |

---

## HTTPステータスコード

| コード | 意味 |
|-------|------|
| 200 | 成功 |
| 201 | 作成成功 (投稿、メッセージ等) |
| 400 | バリデーションエラー |
| 401 | 未認証 / トークン無効 |
| 403 | 権限なし |
| 404 | リソースが見つからない |
| 409 | 競合 (重複操作等) |
| 429 | レート制限超過 |
| 500 | サーバー内部エラー |

---

## レート制限

- ポリシー: `5;w=60` (60秒間に5リクエスト) — エンドポイントにより異なる
- ヘッダー: `ratelimit-policy`, `ratelimit-remaining`, `ratelimit-reset`

---

## 技術スタック

| 項目 | 詳細 |
|------|------|
| Backend | Node.js (Express) + Prisma |
| Database | PostgreSQL |
| Frontend | Vite + React |
| CDN/WAF | Cloudflare |
| リアルタイム | Socket.IO (WebSocketのみ) |
| 認証 | JWT (HS256) + CSRF |
| SSL | Let's Encrypt |

---

## 実装例 (Python)

以下の例では認証済みの `sess` (requests.Session) と `headers` (認証ヘッダー) を使用する前提。

### テキスト投稿

```python
def post_karoot(text):
    r = sess.post(f"{BASE}/posts",
        data={"content": text, "visibility": "PUBLIC", "replyRestriction": "EVERYONE"},
        headers=headers)
    return r.json()["post"]["id"]

post_id = post_karoot("Hello Karotter!")
```

### 画像付き投稿

```python
def post_with_image(text, image_path):
    with open(image_path, "rb") as f:
        files = [("media", (image_path.split("/")[-1], f, "image/png"))]
        data = {
            "content": text,
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
            "mediaAlts": json.dumps([""]),
            "mediaSpoilerFlags": json.dumps([False]),
            "mediaR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/posts", files=files, data=data, headers=headers)
    return r.json()["post"]["id"]
```

### 動画付き投稿

```python
def post_with_video(text, video_path):
    with open(video_path, "rb") as f:
        files = [("media", ("video.mp4", f, "video/mp4"))]
        data = {
            "content": text,
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
            "mediaAlts": json.dumps([""]),
            "mediaSpoilerFlags": json.dumps([False]),
            "mediaR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/posts", files=files, data=data, headers=headers)
    return r.json()["post"]["id"]

# 画像と動画は同時に投稿できない（400エラー）
```

### リプライ

```python
def reply(text, parent_id):
    r = sess.post(f"{BASE}/posts",
        data={
            "content": text,
            "parentId": str(parent_id),
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
        },
        headers=headers)
    return r.json()["post"]["id"]

reply("返信テスト", 12345)
```

### 投稿削除

```python
def delete_post(post_id):
    r = sess.delete(f"{BASE}/posts/{post_id}", headers=headers)
    return r.status_code == 200
```

### タイムライン取得

```python
def get_timeline(page=1, limit=15):
    r = sess.get(f"{BASE}/posts/timeline",
        params={"page": page, "limit": limit, "mode": "latest"},
        headers=headers)
    return r.json()["posts"]

for post in get_timeline():
    print(f"@{post['author']['username']}: {post['content'][:50]}")
```

### 通知取得 & 返信

```python
def get_notifications(limit=30):
    r = sess.get(f"{BASE}/notifications",
        params={"limit": limit},
        headers=headers)
    return r.json()["notifications"]

for notif in get_notifications():
    if notif["type"] == "REPLY":
        post = notif["post"]
        print(f"@{notif['actor']['username']} replied: {post['content']}")
        # reply(f"返信ありがとう", post["id"])
```

### いいね / リアクション

```python
def like(post_id):
    sess.post(f"{BASE}/posts/{post_id}/like", headers=headers)

def unlike(post_id):
    sess.delete(f"{BASE}/posts/{post_id}/like", headers=headers)

def react(post_id, emoji):
    sess.post(f"{BASE}/posts/{post_id}/react",
        json={"emoji": emoji}, headers=headers)

def unreact(post_id, emoji):
    from urllib.parse import quote
    sess.delete(f"{BASE}/posts/{post_id}/react/{quote(emoji)}", headers=headers)

like(12345)
react(12345, "🥕")
```

### フォロー / フォロバ

```python
def follow(user_id):
    sess.post(f"{BASE}/follow/{user_id}", headers=headers)

def get_followers(user_id, limit=100):
    r = sess.get(f"{BASE}/users/{user_id}/followers",
        params={"limit": limit}, headers=headers)
    return r.json()

def get_following(user_id, limit=1000):
    r = sess.get(f"{BASE}/users/{user_id}/following",
        params={"limit": limit}, headers=headers)
    return r.json()

# 自動フォロバ
def auto_followback(my_id):
    followers = {u["id"] for u in get_followers(my_id)}
    following = {u["id"] for u in get_following(my_id)}
    for uid in followers - following:
        follow(uid)
        print(f"Followed back: {uid}")
```

### DM送信

```python
def send_dm(group_id, text):
    r = sess.post(f"{BASE}/dm/groups/{group_id}/messages",
        data={"content": text},
        headers=headers)
    return r.json()

def send_dm_with_image(group_id, text, image_path):
    with open(image_path, "rb") as f:
        files = [("attachments", (image_path.split("/")[-1], f, "image/png"))]
        data = {
            "content": text,
            "attachmentAlts": json.dumps([""]),
            "attachmentSpoilerFlags": json.dumps([False]),
            "attachmentR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/dm/groups/{group_id}/messages",
            files=files, data=data, headers=headers)
    return r.json()

# 投稿のフィールド名は "media"、DMは "attachments" — 間違えると500エラー
```

### DM取得

```python
def get_dm_groups():
    r = sess.get(f"{BASE}/dm/groups", headers=headers)
    return r.json()["groups"]

def get_dm_messages(group_id, limit=50):
    r = sess.get(f"{BASE}/dm/groups/{group_id}/messages",
        params={"limit": limit}, headers=headers)
    return r.json()["messages"]
```

### 投票

```python
def vote(post_id, option_id):
    sess.post(f"{BASE}/posts/{post_id}/poll/vote",
        json={"optionId": option_id}, headers=headers)

# 投票データは GET /posts/{id} の post.poll に含まれる
# 再度POSTで投票取り消し（トグル式）
```

### Developer API (APIキー認証)

```python
API_KEY = "kar_live_xxxxx"
DEV_BASE = "https://api.karotter.com/api/developer"
dev_headers = {
    "Authorization": f"Bearer {API_KEY}",
    "X-Client-Type": "web",
    "X-Device-Id": "550e8400-e29b-41d4-a716-446655440000"
}

# タイムライン取得
r = requests.get(f"{DEV_BASE}/timeline", headers=dev_headers)
posts = r.json()["posts"]

# 投稿
r = requests.post(f"{DEV_BASE}/posts",
    json={"content": "Developer APIから投稿"},
    headers=dev_headers)

# いいね
requests.post(f"{DEV_BASE}/posts/12345/like", headers=dev_headers)

# Developer APIでは DELETE /posts/{id} (投稿削除) は使用不可
```
