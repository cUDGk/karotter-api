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
  "message": "ログインに成功しました",
  "accessToken": "eyJ...",
  "sessionId": "uuid",
  "deviceId": "uuid",
  "user": {
    "id": 15459,
    "username": "claude",
    "displayName": "claude",
    "email": "",
    "avatarUrl": "/uploads/avatars/...",
    "emailVerified": false
  }
}
```
- `accessToken`: JWT (HS256), 30分有効
- Set-Cookie: `karotter_at` (HttpOnly, Secure, SameSite=Strict, 1h), `karotter_rt` (HttpOnly, Secure, 30d), `karotter_csrf` (Secure, 30d)
- レート制限: 5 req / 60s

### CSRFトークン
```
GET    /auth/csrf-token     → トークン取得
DELETE /auth/csrf-token     → トークン無効化

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

Response 200: {"message": "ログアウトしました"}
```
- 3つのCookieをクリア (karotter_at, karotter_rt, karotter_csrf)

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
POST /auth/resend-verification    → メール認証再送 (600秒クールダウン)
POST /auth/me/email               → メールアドレス変更
```

### セッション管理
```
GET  /auth/sessions                  → 全セッション一覧
DELETE /auth/sessions/:id            → 特定セッション削除
DELETE /auth/sessions/others         → 他の全セッションを削除
DELETE /auth/sessions/all            → 全セッション削除
POST /auth/switch-session            → セッション切り替え (マルチアカウント用)
  Body: {"sessionId": "uuid"} または {"userId": 12345}
  Response 200: {"accessToken": "jwt...", "sessionId": "uuid", "user": {...}}
  ※ 新しいアクセストークンとセッションIDが発行される
  ※ 空Body: 400 {"error": "セッションIDまたはユーザーIDが必要です"}
  ※ 無効なセッション: 401 {"error": "無効なセッションです"}
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
  "post": {
    "id": 12345,
    "content": "テキスト",
    "authorId": 15459,
    "parentId": null,
    "quotedPostId": null,
    "mediaUrls": [],
    "mediaTypes": [],
    "mediaAlts": [],
    "mediaSpoilerFlags": [],
    "mediaR18Flags": [],
    "isR18": false,
    "hideFromMinors": false,
    "adminForceHidden": false,
    "adminForceR18": false,
    "embedUrl": null,
    "embedTitle": null,
    "embedDescription": null,
    "embedImage": null,
    "likesCount": 0,
    "rekarotsCount": 0,
    "repliesCount": 0,
    "viewsCount": 0,
    "bookmarksCount": 0,
    "replyRestriction": "EVERYONE",
    "visibility": "PUBLIC",
    "isAiGenerated": false,
    "isPromotional": false,
    "bookmarked": false,
    "createdAt": "2026-03-29T...",
    "author": {
      "id": 15459,
      "username": "claude",
      "displayName": "claude",
      "avatarUrl": "/uploads/avatars/...",
      "avatarFrameId": null,
      "officialMark": [],
      "isBotAccount": true,
      "isParodyAccount": false,
      "adminForceBot": true,
      "adminForceParody": false,
      "isPrivate": false
    },
    "poll": null
  }
}
```
- レート制限: 50 req / 3600s
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

Response 200: {"message": "カロートを削除しました"}
```
- 他人の投稿: `403 {"error": "このカロートを削除する権限がありません"}`
- 削除後のGET: `404 {"error": "カロートが見つかりません"}`

### 投稿の返信一覧
```
GET /posts/{id}/replies
```

### いいねしたユーザー一覧
```
GET /posts/{id}/likes
```

### 引用一覧
```
GET /posts/{id}/quotes
```

### リカロートしたユーザー一覧
```
GET /posts/{id}/rekarots
```

### 投稿のリアクション一覧
```
GET /posts/{id}/reactions
```

### 会話スレッド
```
GET  /posts/{id}/conversation        → 会話メタデータ (参加者等)
POST /posts/{id}/conversation/leave  → 会話から離脱
```

### 投稿アナリティクス
```
GET /posts/{id}/analytics

Response 200: (自分の投稿のみ、他人は403)
{
  "viewsCount": 100,
  "likesCount": 6,
  "rekarotsCount": 0,
  "repliesCount": 1,
  "audience": {
    "gender": {"MALE": 30, "FEMALE": 40, "OTHER": 30},
    "age": {"13-17": 10, "18-24": 50, "25-34": 30, "35-44": 5, "45-54": 3, "55+": 1, "unknown": 1}
  },
  "knownViewerCount": 80,
  "anonymousViews": 20
}
```

### タイムライン
```
GET /posts/timeline?page=1&limit=15&mode=latest
```
- `mode`: `latest` (最新), `trending` (トレンド), `following` (フォロー中)

```
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

### 予約投稿
```
GET    /posts/scheduled/me       → 一覧
DELETE /posts/scheduled/{id}     → 予約投稿削除
```

### 自分の返信
```
GET /posts/me/replies
```

### 閲覧数バッチ送信
```
POST /posts/batch-views
```

### ベータUIフィードバック
```
POST /posts/feedback/beta-survey
Content-Type: application/json

{
  "preference": "beta" | "current"
}
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

Response 200: {"message": "いいねしました"}
```
- レート制限: 30 req / 60s
- 重複: `400 {"error": "既にいいねしています"}`

### いいね取り消し
```
DELETE /posts/{id}/like

Response 200: {"message": "いいねを取り消しました"}
```
- `/likes` (複数形) は404。単数形のみ

---

## リアクション (Reaction)

### リアクション追加
```
POST /posts/{id}/react
Content-Type: application/json

{
  "emoji": "👍"
}

Response 200: {"message": "リアクションしました"}
```
- 絵文字は32文字以内 (任意文字列可)
- 重複: `400 {"error": "既にこの絵文字でリアクションしています"}`
- レート制限: 30 req / 60s

### リアクション取り消し
```
DELETE /posts/{id}/react/{url-encoded-emoji}

Response 200: {"message": "リアクションを取り消しました"}
```
- 絵文字はURLエンコード必須 (e.g. `%F0%9F%91%8D` = 👍)

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

Response 200: {"message": "フォローしました"}
```
- レート制限: 5 req / 60s (厳しい)
- 重複: `400 {"error": "既にフォローしています"}`
- 自分自身: `400`

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

Response 200: {"message": "フォローを解除しました"}
```

### フォロワー削除
```
DELETE /follow/follower/{userId}    → フォロワーを外す
```

### フォローリクエスト
```
GET  /follow/requests/pending          → 保留中の受信リクエスト
POST /follow/requests/{id}/accept      → 承認
POST /follow/requests/{id}/reject      → 拒否
```

### ブロック
```
GET    /follow/block              → ブロック一覧
POST   /follow/block/{userId}     → ブロック追加
DELETE /follow/block/{userId}     → ブロック解除
```

### ミュート
```
GET    /follow/mute               → ミュート一覧
POST   /follow/mute/{userId}      → ミュート追加
DELETE /follow/mute/{userId}      → ミュート解除
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

### ユーザーのいいねした投稿
```
GET /users/{userId}/likes
```

### ユーザーのメディア投稿
```
GET /users/{userId}/media
```

### ユーザーの返信一覧
```
GET /users/{userId}/replies
```

### 相互フォロワー
```
GET /users/{userId}/mutual-followers
```

### おすすめユーザー
```
GET /users/recommended?limit=3
```

### ユーザー名クォータ確認
```
GET /users/username/quota

Response 200:
{
  "windowDays": 14,
  "maxChanges": 5,
  "usedChanges": 0,
  "remainingChanges": 5
}
```

### プロフィール更新
```
PATCH /users/profile
Content-Type: application/json

{
  "displayName": "表示名",
  "bio": "自己紹介",
  "websiteUrl": "https://example.com",
  "location": "東京",
  "birthday": "2000-01-01",
  "birthdayVisibility": "PRIVATE",
  "gender": "OTHER"
}

Response 200:
{
  "message": "プロフィールを更新しました",
  "user": { ... }
}
```
- 変更したいフィールドのみ送信可 (空オブジェクトでも200を返す)

### ステータス更新
```
PATCH /users/status
```

### パスワード変更
```
PATCH /users/password
```

### ユーザー名変更
```
PATCH /users/username
```

### ピン留め投稿変更
```
PATCH /users/profile/pinned-post
```

### ユーザー設定
```
PATCH /users/settings
```

### アカウント削除
```
DELETE /users/account
```

### アバター / ヘッダー更新
```
POST /profile/avatar    → アバター (multipart/form-data)
POST /profile/header    → ヘッダー画像 (multipart/form-data)
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
- `type`: `REPLY`, `MENTION`, `FOLLOW`, `FOLLOW_REQUEST`, `LIKE`, `REKAROT`, `QUOTE`, `DM`

### 未読通知数
```
GET /notifications/unread/count
```

### グループ化された投稿通知
```
GET /notifications/grouped-posts
```

### 全通知既読
```
PATCH /notifications/read-all
```

### 通知削除
```
DELETE /notifications/{id}        → 個別削除
DELETE /notifications/all         → 全削除
```

### プッシュ通知登録
```
POST /notifications/push/register
POST /notifications/push/unregister
```

---

## リカロート (Rekarot)

### リカロートする / 取り消す
```
POST   /posts/{id}/rekarot    → {"message": "リカロートしました"}
DELETE /posts/{id}/rekarot    → {"message": "リカロートを取り消しました"}
```
- レート制限: 30 req / 60s

---

## ブックマーク

### ブックマーク一覧
```
GET /posts/me/bookmarks
Authorization: Bearer {token}
```

### ブックマーク追加 / 解除
```
POST   /posts/{id}/bookmark    → {"message": "ブックマークしました"}
DELETE /posts/{id}/bookmark    → {"message": "ブックマークを解除しました"}
```
- レート制限: 30 req / 60s

---

## DM (ダイレクトメッセージ)

### DMグループ一覧 / 作成
```
POST /dm/groups                → グループ作成 (Body: {"userIds": [userId, ...]})
GET  /dm/groups                → グループ一覧
```
グループ内フィールド: `id`, `members[]` (オンライン状態含む), `lastMessage`, `activeCall`, `canSend`, `sendDisabledReason`

```
GET  /dm/groups
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

### 1対1 DM開始
```
POST /dm/start
Content-Type: application/json

{
  "targetUserId": 12345
}
```

### グループ操作
```
POST   /dm/groups/{groupId}/clear              → チャット履歴クリア (自分の表示のみ削除)
POST   /dm/groups/{groupId}/leave              → グループ退出 (1対1DMは退出不可、クリアを使用)
POST   /dm/groups/{groupId}/members            → メンバー追加 (メール認証必須)
DELETE /dm/groups/{groupId}/members/{userId}    → メンバー削除
POST   /dm/groups/{groupId}/request/accept     → DMリクエスト承認
POST   /dm/groups/{groupId}/request/reject     → DMリクエスト拒否
```

### メッセージ操作
```
DELETE /dm/messages/{id}              → メッセージ削除
PATCH  /dm/messages/{id}              → メッセージ編集
POST   /dm/messages/{id}/reactions    → メッセージリアクション
POST   /dm/messages/{id}/poll/vote    → DM内投票
```

### 通話
```
GET  /dm/groups/{groupId}/call          → 通話状態確認
POST /dm/groups/{groupId}/call/start    → 通話開始
POST /dm/groups/{groupId}/call/join     → 通話参加
POST /dm/groups/{groupId}/call/leave    → 通話退出
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

### トレンドハッシュタグ
```
GET /search/trending/hashtags
```

### ディスカバー
```
GET /search/discover/latest?limit=12&cursor={lastPostId}    → 最新投稿
GET /search/discover/media?limit=12&cursor={lastPostId}     → メディア投稿
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

### 招待リンクのローテーション
```
POST /draw/rooms/{roomId}/invite/rotate
```

---

## ラジオ (Radio)

### ラジオ作成
```
POST /radio
Content-Type: application/json

{
  "title": "ラジオタイトル"
}
```
- タイトルは1〜100文字

### アクティブなラジオ一覧
```
GET /radio/active
```

### ラジオ詳細
```
GET /radio/{id}
```

### ラジオ内メッセージ
```
GET  /radio/{id}/messages    → メッセージ一覧
POST /radio/{id}/messages    → メッセージ送信
```

### ラジオ設定
```
PATCH /radio/{id}/settings
```

### 参加者管理
```
PATCH  /radio/{id}/participants/{userId}/role            → ロール変更
PATCH  /radio/{id}/participants/{userId}/mute            → ミュート
POST   /radio/{id}/participants/{userId}/invite-speaker  → スピーカー招待
DELETE /radio/{id}/participants/{userId}/invite-speaker  → スピーカー招待取消
POST   /radio/{id}/accept-speaker-invite                → スピーカー招待承諾
POST   /radio/{id}/request-speaker                      → スピーカーリクエスト
```

### ラジオ参加 / 退出 / 終了
```
POST /radio/{id}/join
POST /radio/{id}/leave
POST /radio/{id}/end
```

---

## ソーシャル

### サークル
```
GET    /social/circles                        → 一覧
POST   /social/circles                        → 作成
DELETE /social/circles/{id}                   → 削除
POST   /social/circles/{id}/members           → メンバー追加
DELETE /social/circles/{id}/members/{userId}  → メンバー削除
```

### リスト
```
GET    /social/lists                          → 一覧
POST   /social/lists                          → 作成
DELETE /social/lists/{id}                     → 削除
POST   /social/lists/{id}/members             → メンバー追加
DELETE /social/lists/{id}/members/{userId}    → メンバー削除
GET    /social/lists/{id}/posts               → リストの投稿一覧
```

### ストーリー
```
GET    /social/stories                → ストーリー一覧
POST   /social/stories                → ストーリー投稿 (画像/動画)
DELETE /social/stories/{id}           → ストーリー削除
GET    /social/stories/user/{userId}  → ユーザーのストーリー
GET    /social/stories/{id}/viewers   → 閲覧者一覧
POST   /social/stories/{id}/views    → 閲覧記録
POST   /social/stories/{id}/like     → ストーリーいいね
DELETE /social/stories/{id}/like     → ストーリーいいね取消
```

### リンクプレビュー
```
GET /social/link-preview?url={url}        → OGP情報取得
GET /social/link-preview-image?url={url}  → プレビュー画像取得 (常時500の可能性)
```

---

## レポート / お問い合わせ

### 通報
```
POST /reports
Content-Type: application/json

{
  "type": "POST",       // 通報対象種類
  "postId": 12345,      // 投稿通報時
  "reason": "理由"
}
```
- `type` が不正: `400 {"error": "通報対象の種類が不正です"}`
- レート制限: 10 req / 900s

### お問い合わせ
```
POST /contact
Content-Type: application/json

{
  "category": "bug" | "feature_request" | "general_inquiry",
  "subject": "件名",
  "body": "本文"
}
```
- 認証不要
- レート制限: 5 req / 3600s

---

## 規約

```
GET /legal/summary    → 規約バージョン・発効日
GET /legal/terms      → 利用規約全文
GET /legal/privacy    → プライバシーポリシー全文
GET /legal/{slug}     → 法的文書（汎用、認証不要）
```

レスポンス例 (`/legal/summary`):
```json
{
  "version": "terms:2026-03-27|privacy:2026-03-27",
  "termsEffectiveDate": "2026-03-27",
  "privacyEffectiveDate": "2026-03-27"
}
```

---

## 埋め込み (Embed)

```
GET /embed/{postId}     → 投稿の埋め込みHTML
GET /oembed?url={url}   → oEmbed形式
```

---

## 設定

```
GET  /settings           → 全設定取得
GET  /settings/{key}     → 個別設定取得
```

---

## 管理パネル (Admin)

### 概要

| 項目 | 詳細 |
|------|------|
| フロントエンドURL | `https://karotter.com/admin` |
| APIベースパス | `/control-room-x9k2` |
| JSチャンク | `Admin-BX1OyPEg.js` |
| 認証 | `user.isAdmin === true` が必要 |
| 非管理者アクセス | 「アクセス拒否 — 管理者権限が必要です」表示 |
| 全APIレスポンス | `403: 管理者権限が必要です` |
| 確認済みエンドポイント | 136 |

### UIタブ構成 (JSチャンクから確認)

| タブID | ラベル | 説明 |
|--------|--------|------|
| `dashboard` | ダッシュボード | システム統計 (総ユーザー数、総カロート数、24h活動ユーザー、未対応通報) |
| `users` | ユーザー管理 | 検索、BAN/解除、フラグ変更、公式マーク付与、アカウント編集、削除 |
| `posts` | カロート管理 | 検索、R18強制、非表示、削除 |
| `stories` | ストーリー管理 | 検索、R18強制、非表示、削除、有効期限表示 |
| `recommend` | おすすめ β | おすすめアルゴリズムテスト (userId指定、スコア詳細表示) |
| `trending` | トレンド β | トレンドアルゴリズムテスト (velocity, uniqueActors, engagementRate) |
| `survey` | アンケート | おすすめ満足度アンケート結果 (投票数、満足度スコア) |
| `beta-experiment` | Beta実験 | A/Bテスト結果 (A〜E 5バリアント、userId%5で振り分け) |

### ダッシュボード統計

```
GET /control-room-x9k2/analytics

Response:
{
  "totalUsers": 12000,
  "totalPosts": 120000,
  "activeUsers": 2300,     // 24時間アクティブ
  "totalReports": 5        // 未対応通報数
}
```

### ユーザー管理

管理者がユーザーに対して行える操作:

**アカウント情報変更** (`PATCH /control-room-x9k2/users/{id}/account`):
- username、displayName の変更
- email 変更（任意）
- password 強制リセット（任意）
- emailVerified フラグの切り替え

**公式マーク** (`PATCH /control-room-x9k2/users/{id}/official-mark`):

| 値 | ラベル |
|-----|--------|
| `YELLOW` | 黄色: 認証済み団体マーク |
| `PURPLE` | 紫: 運営マーク |
| `BLUE` | 青: 本人マーク |
| `GRAY` | 灰色: 政府関係者マーク |
| `BLACK` | 黒: 認証済みマーク |
| `RED` | 赤: 認証済みマーク |
| `GREEN` | 緑: 認証済みマーク |
| `ORANGE` | オレンジ: 認証済みマーク |

- 複数マーク同時付与可能（配列で管理）

**フラグ操作** (`PATCH /control-room-x9k2/users/{id}/flags`):

| フラグ | 説明 |
|--------|------|
| `isParodyAccount` | パロディアカウント |
| `isBotAccount` | BOTユーザー |
| `adminForceHidden` | 管理者非表示 |
| `adminForceParody` | パロディ強制 |
| `adminForceBot` | BOT強制 |
| `showBotAccounts` | BOT表示許可 |
| `showHiddenPosts` | 非表示閲覧許可 |
| `showR18Content` | R18表示許可 |
| `hideProfileFromMinors` | 未成年にプロフ非表示 |

**BAN** (`PATCH /control-room-x9k2/users/{id}/ban`):
- BAN時に `reason` を指定
- BAN中ユーザーには赤い「BAN中」バッジ表示
- `unban` で解除

**削除** (`DELETE /control-room-x9k2/users/{id}`):
- 確認ダイアログ:「@{username} を削除しますか？この操作は取り消せません。」

### 投稿管理

管理者が投稿に対して行える操作:
- `adminForceR18`: R18強制フラグ
- `adminForceHidden`: 非表示フラグ
- 投稿削除

表示情報: viewsCount、likesCount、rekarotsCount、repliesCount

### ストーリー管理

投稿管理と同様のフラグ操作に加え:
- mediaType（画像/動画）のプレビュー表示
- caption表示
- visibility（PUBLIC/FOLLOWERS/サークル）
- 有効期限（expiresAt）の表示

### おすすめアルゴリズムテスト (β)

```
GET /control-room-x9k2/test-recommend?limit=30&userId={id}

Response:
{
  "targetUser": { "id", "username", "displayName" },
  "followingCount": 100,
  "candidateCount": 500,
  "signalsSummary": {
    "authorAffinityCount": 50,
    "socialProofCount": 30,
    "mutualFollowCount": 20,
    "tagAffinityTop10": [{"tag": "プログラミング", "score": 85}]
  },
  "rawTopByScore": [{"rank", "author", "score", "trending"}],
  "ranked": [{
    "rank", "author", "content",
    "stats": {"likes", "rekarots", "replies", "views"},
    "ageHours",
    "scores": {
      "engagement", "freshness", "authorAffinity",
      "graph", "socialProof", "tagAffinity",
      "inNetwork", "totalRecommend"
    }
  }]
}
```

**A/Bテスト 5バリアント**:

| タイプ | コンセプト | 新しさ | エンゲージメント | 会話 | 発見 |
|--------|-----------|--------|----------------|------|------|
| A | 新しさ最優先 | ★★★ | ★ | - | - |
| B | 質重視 | ★ | ★★★ | ★ | - |
| C | バランス | ★★ | ★★ | ★ | ★ |
| D | 会話重視 | ★★ | ★★ | ★★★ | - |
| E | 発見重視 | ★★ | ★★ | ★ | ★★★ |

ユーザーは `userId % 5` でバリアントに自動振り分け。

### トレンドアルゴリズムテスト (β)

```
GET /control-room-x9k2/test-trending?limit=30

Response:
{
  "candidateCount": 200,
  "rawTop10": [{"rank", "author", "total", "velocity", "uniqueActors"}],
  "ranked": [{
    "rank", "author", "content",
    "stats": {"likes", "rekarots", "replies", "views"},
    "ageHours",
    "trendBreakdown": {
      "velocity", "uniqueActors", "engagementRate", "freshness", "total"
    },
    "windowStats": {
      "likes1h", "likes6h", "likes24h",
      "rekarots1h", "rekarots6h", "rekarots24h",
      "replies1h", "replies6h", "replies24h",
      "uniqueActors"
    }
  }]
}
```

### コア (3)
```
GET /control-room-x9k2
GET /control-room-x9k2/dashboard
GET /control-room-x9k2/overview
```

### ユーザー管理 (22)
```
GET    /control-room-x9k2/users
GET    /control-room-x9k2/users/{id}
GET    /control-room-x9k2/users/search
POST   /control-room-x9k2/users
PATCH  /control-room-x9k2/users/{id}/ban
PATCH  /control-room-x9k2/users/{id}/unban
PATCH  /control-room-x9k2/users/{id}/verify
PATCH  /control-room-x9k2/users/{id}/flags
PATCH  /control-room-x9k2/users/{id}/account
PATCH  /control-room-x9k2/users/{id}/official-mark
PATCH  /control-room-x9k2/users/{id}/email
PATCH  /control-room-x9k2/users/{id}/password
PATCH  /control-room-x9k2/users/{id}/role
GET    /control-room-x9k2/users/{id}/sessions
GET    /control-room-x9k2/users/{id}/posts
GET    /control-room-x9k2/users/{id}/reports
GET    /control-room-x9k2/users/{id}/bans
GET    /control-room-x9k2/users/{id}/notes
GET    /control-room-x9k2/users/{id}/history
GET    /control-room-x9k2/users/{id}/suspend
GET    /control-room-x9k2/users/{id}/restrict
POST   /control-room-x9k2/users/{id}/warn
DELETE /control-room-x9k2/users/{id}/delete
```

### 投稿管理 (6)
```
GET    /control-room-x9k2/posts
GET    /control-room-x9k2/posts/{id}
GET    /control-room-x9k2/posts/search
PATCH  /control-room-x9k2/posts/{id}/flags
PATCH  /control-room-x9k2/posts/{id}/hide
DELETE /control-room-x9k2/posts/{id}/delete
```

### ストーリー管理 (2)
```
GET    /control-room-x9k2/stories
PATCH  /control-room-x9k2/stories/{id}/flags
```

### 通報 / モデレーション (11)
```
GET  /control-room-x9k2/reports
GET  /control-room-x9k2/reports/pending
GET  /control-room-x9k2/reports/resolved
GET  /control-room-x9k2/reports/{id}
POST /control-room-x9k2/reports/{id}/resolve
POST /control-room-x9k2/reports/{id}/dismiss
POST /control-room-x9k2/reports/{id}/escalate
GET  /control-room-x9k2/moderation
GET  /control-room-x9k2/moderation/queue
GET  /control-room-x9k2/moderation/filters
GET  /control-room-x9k2/moderation/automod
GET  /control-room-x9k2/moderation/words
GET  /control-room-x9k2/moderation/rules
GET  /control-room-x9k2/flagged-content
```

### BAN (6)
```
GET  /control-room-x9k2/bans
GET  /control-room-x9k2/bans/{id}
POST /control-room-x9k2/bans/create
GET  /control-room-x9k2/ip-bans
GET  /control-room-x9k2/ip-bans/{id}
GET  /control-room-x9k2/shadowbans
```

### コンテンツ管理 (15)
```
GET  /control-room-x9k2/badges
GET  /control-room-x9k2/badges/{id}
POST /control-room-x9k2/badges/create
GET  /control-room-x9k2/emoji
GET  /control-room-x9k2/emoji/{id}
POST /control-room-x9k2/emoji/create
GET  /control-room-x9k2/frames
GET  /control-room-x9k2/frames/{id}
GET  /control-room-x9k2/gacha
GET  /control-room-x9k2/gacha/{id}
GET  /control-room-x9k2/gacha/items
GET  /control-room-x9k2/themes
GET  /control-room-x9k2/themes/{id}
GET  /control-room-x9k2/stickers
GET  /control-room-x9k2/stickers/{id}
```

### お知らせ (3)
```
GET  /control-room-x9k2/announcements
GET  /control-room-x9k2/announcements/{id}
POST /control-room-x9k2/announcements/create
```

### アナリティクス (11)
```
GET /control-room-x9k2/analytics
GET /control-room-x9k2/analytics/users
GET /control-room-x9k2/analytics/posts
GET /control-room-x9k2/analytics/growth
GET /control-room-x9k2/analytics/engagement
GET /control-room-x9k2/analytics/retention
GET /control-room-x9k2/analytics/revenue
GET /control-room-x9k2/stats
GET /control-room-x9k2/stats/users
GET /control-room-x9k2/stats/posts
GET /control-room-x9k2/stats/daily
```

### 申請 / リクエスト (3)
```
GET /control-room-x9k2/verification-requests
GET /control-room-x9k2/bot-requests
GET /control-room-x9k2/appeals
```

### システム (16)
```
GET  /control-room-x9k2/settings
POST /control-room-x9k2/settings
GET  /control-room-x9k2/config
GET  /control-room-x9k2/system
GET  /control-room-x9k2/logs
GET  /control-room-x9k2/logs/{type}
GET  /control-room-x9k2/audit
GET  /control-room-x9k2/audit-log
GET  /control-room-x9k2/maintenance
POST /control-room-x9k2/maintenance
GET  /control-room-x9k2/backup
POST /control-room-x9k2/backup
GET  /control-room-x9k2/cron
GET  /control-room-x9k2/queue
GET  /control-room-x9k2/tasks
GET  /control-room-x9k2/jobs
GET  /control-room-x9k2/cache
POST /control-room-x9k2/cache/clear
GET  /control-room-x9k2/database
GET  /control-room-x9k2/migrations
```

### その他管理 (38)
```
GET /control-room-x9k2/beta-experiment
GET /control-room-x9k2/test-recommend
GET /control-room-x9k2/test-trending
GET /control-room-x9k2/survey-results
GET /control-room-x9k2/feedback
GET /control-room-x9k2/media
GET /control-room-x9k2/uploads
GET /control-room-x9k2/notifications
GET /control-room-x9k2/invites
GET /control-room-x9k2/features
GET /control-room-x9k2/feature-flags
GET /control-room-x9k2/roles
GET /control-room-x9k2/permissions
GET /control-room-x9k2/radio
GET /control-room-x9k2/draw
GET /control-room-x9k2/dm
GET /control-room-x9k2/emails
GET /control-room-x9k2/actions
GET /control-room-x9k2/sessions
GET /control-room-x9k2/webhooks
GET /control-room-x9k2/api-keys
GET /control-room-x9k2/apikeys
GET /control-room-x9k2/premium
GET /control-room-x9k2/subscriptions
GET /control-room-x9k2/monetization
GET /control-room-x9k2/payments
GET /control-room-x9k2/domains
GET /control-room-x9k2/rate-limits
GET /control-room-x9k2/blocked-words
GET /control-room-x9k2/filtered-words
GET /control-room-x9k2/trending
GET /control-room-x9k2/trending/override
GET /control-room-x9k2/search
GET /control-room-x9k2/search/index
```

---

## APIキー

```
GET    /apikeys        → APIキー一覧
POST   /apikeys        → APIキー作成
DELETE /apikeys/{id}   → APIキー削除
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
| GET | `/notifications` | 通知一覧 |
| GET | `/users/{id}` | ユーザー情報 |
| POST | `/posts/{id}/like` | いいね |
| DELETE | `/posts/{id}/like` | いいね取消 |
| POST | `/posts/{id}/bookmark` | ブックマーク追加 |
| POST | `/posts/{id}/rekarot` | リカロート |
| DELETE | `/posts/{id}/rekarot` | リカロート取消 |

### Developer APIで利用不可

以下はセルフBot API (`/api/`) でのみ利用可能:

- `DELETE /posts/{id}` (投稿削除)
- `/auth/*` (認証関連)
- `/dm/*` (ダイレクトメッセージ)
- `/follow/*` (フォロー操作)
- `/draw/*` (絵チャ)
- `/radio/*` (ラジオ)

### 備考
- APIキーは `kar_live_` プレフィックスの形式
- `/apikeys` エンドポイントでAPIキーを管理可能 (GET/POST/DELETE)
- Developer APIは認証ミドルウェアがルートマッチング前に実行されるため、無効なキーでは全パスが401を返す
- 認証系は現在リライト中の可能性あり

---

## Socket.IO (リアルタイム通信)

**接続先**: `wss://karotter.com/socket.io`

### DM
| イベント | 方向 | 説明 |
|---------|------|------|
| `dd.join` | C→S | DMグループ参加 |
| `dd.leave` | C→S | DMグループ退出 |
| `dd.read` | C→S | 既読マーク |
| `dd.newMessage` | S→C | 新規メッセージ受信 |
| `dd.messageDeleted` | S→C | メッセージ削除 |
| `dd.messageUpdated` | S→C | メッセージ更新 |
| `dd.memberAdded` | S→C | メンバー追加 |
| `dd.memberLeft` | S→C | メンバー退出 |
| `dd.memberRemoved` | S→C | メンバー削除 |
| `dd.requestUpdated` | S→C | DMリクエスト更新 |

### 通話 (Voice)
| イベント | 方向 | 説明 |
|---------|------|------|
| `voice:offer` | C→S | WebRTC SDP Offer |
| `voice:answer` | C→S | WebRTC SDP Answer |
| `voice:ice-candidate` | C→S | ICE candidate |
| `voice:hangup` | C→S | 通話切断 |
| `voice:participant-state` | C→S | マイク/スピーカー状態 |
| `fd.offer` | S→C | SDP Offer受信 |
| `fd.answer` | S→C | SDP Answer受信 |
| `fd.iceCandidate` | S→C | ICE candidate受信 |
| `fd.hangup` | S→C | 切断通知 |
| `fd.participantState` | S→C | 参加者状態変更 |
| `pd.state` | S→C | 通話状態変更 |
| `pd.incoming` | S→C | 着信通知 |

### スクリーンシェア
| イベント | 方向 | 説明 |
|---------|------|------|
| `screen-share:view` | C→S | 画面共有視聴 |

### ラジオ/スペース
| イベント | 方向 | 説明 |
|---------|------|------|
| `radio:signal` | C→S | WebRTCシグナリング |
| `radio:renegotiate-request` | C→S | 再ネゴシエーション |
| `radio:participant-state` | C→S | 参加者状態 |
| `vd.join` | S→C | スペース参加通知 |
| `vd.leave` | S→C | スペース退出通知 |
| `vd.userJoined` | S→C | ユーザー参加 |
| `vd.userLeft` | S→C | ユーザー退出 |
| `vd.ended` | S→C | スペース終了 |
| `vd.signal` | S→C | WebRTCシグナル受信 |

### その他
| イベント | 方向 | 説明 |
|---------|------|------|
| `notification` | S→C | プッシュ通知 |
| `hd.status` | S→C | ユーザーオンライン状態変更 |
| `draw:join` | C→S | 絵チャルーム参加 |
| `draw:leave` | C→S | 絵チャルーム退出 |
| `draw:stroke` | C↔S | 描画ストローク |
| `draw:cursor` | C↔S | カーソル位置 |
| `draw:layer-sync` | C↔S | レイヤー同期 |
| `draw:chat` | C↔S | 絵チャ内チャット |
| `draw:room-state` | S→C | 絵チャルーム状態 |
| `draw:user-left` | S→C | ユーザー退出通知 |
| `draw:error` | S→C | エラー通知 |
| `radio:message` | C↔S | スペース内メッセージ |
| `radio:reaction` | C↔S | スペース内リアクション |

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

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `id` | number | 投稿ID |
| `content` | string | 本文 (メンション・ハッシュタグ含む) |
| `authorId` | number | 投稿者ID |
| `parentId` | number\|null | 返信先投稿ID |
| `quotedPostId` | number\|null | 引用元投稿ID |
| `createdAt` | string | 投稿日時 (ISO 8601) |
| `editedAt` | string\|null | 編集日時 |
| `mediaUrls` | string[] | メディアの相対パス |
| `mediaTypes` | string[] | `"image"` / `"video"` |
| `mediaAlts` | string[] | Alt text |
| `mediaSpoilerFlags` | boolean[] | スポイラーフラグ |
| `mediaR18Flags` | boolean[] | R18フラグ |
| `visibility` | string | `PUBLIC` / `FOLLOWERS` / `CIRCLE` / `PRIVATE` |
| `replyRestriction` | string | `EVERYONE` / `FOLLOWERS` / `CIRCLE` / `MENTIONS` |
| `isAiGenerated` | boolean | AI生成フラグ |
| `isPromotional` | boolean | プロモーションフラグ |
| `isR18` | boolean | R18フラグ |
| `hideFromMinors` | boolean | 未成年非表示 |
| `adminForceHidden` | boolean | 管理者非表示 |
| `adminForceR18` | boolean | 管理者R18強制 |
| `likesCount` | number | いいね数 |
| `rekarotsCount` | number | リカロート数 |
| `repliesCount` | number | 返信数 |
| `viewsCount` | number | 閲覧数 |
| `bookmarksCount` | number | ブックマーク数 |
| `reactionsCount` | number | リアクション数 |
| `liked` | boolean | 自分がいいねしたか |
| `rekaroted` | boolean | 自分がリカロートしたか |
| `bookmarked` | boolean | 自分がブックマークしたか |
| `isMutedByViewer` | boolean | ミュート中か |
| `hasBlockedAuthor` | boolean | 投稿者をブロック中か |
| `isBlockedByAuthor` | boolean | 投稿者にブロックされてるか |
| `canInteract` | boolean | インタラクション可能か |
| `canQuote` | boolean | 引用可能か |
| `author` | object | 投稿者情報 |
| `poll` | object\|null | 投票データ |
| `reactions` | array | リアクション配列 |
| `reactionSummary` | array | リアクション集計 |
| `mentions` | array | メンション |
| `hashtags` | array | ハッシュタグ |

### タイムライン固有フィールド

タイムラインの各アイテムには追加フィールドあり:
| フィールド | 説明 |
|-----------|------|
| `type` | `"POST"` / `"REKAROT"` |
| `time` | アイテムの表示時刻 |
| `itemId` | タイムラインアイテムID |

### 通知グルーピング

通知はグルーピングされて返される:
| フィールド | 説明 |
|-----------|------|
| `groupKey` | グループキー |
| `actors[]` | アクションしたユーザー配列 |
| `actorCount` | アクターの総数 |
| `posts[]` | 関連投稿配列 |
| `postCount` | 投稿の総数 |
| `notificationIds[]` | 通知ID配列 |

### snake_case 例外

検索ユーザー結果のみ `is_following` (snake_case)。他は全てcamelCase。

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

レスポンスヘッダー: `ratelimit-policy`, `ratelimit-limit`, `ratelimit-remaining`, `ratelimit-reset`

| エンドポイント | 制限 |
|--------------|------|
| POST /auth/login | 5 req / 60s |
| POST /follow/{id} | 5 req / 60s |
| POST /contact | 5 req / 3600s |
| POST /reports | 10 req / 900s |
| PATCH /users/profile | 15 req / 60s |
| POST /posts/{id}/like, /react, /bookmark, /rekarot | 30 req / 60s |
| POST /posts | 50 req / 3600s |
| その他全般 | 100 req / 60s |

## エラーレスポンス形式

```json
// バリデーション・ビジネスエラー
{"error": "メッセージ", "status": 400}

// 認証エラー
{"error": "無効なトークンです"}

// 権限エラー
{"error": "このカロートを削除する権限がありません", "status": 403}

// 管理者権限
{"error": "管理者権限が必要です", "status": 403}
```

---

## Cookie / LocalStorage

### Cookie
| 名前 | 説明 |
|------|------|
| `karotter_at` | アクセストークン |
| `karotter_rt` | リフレッシュトークン |
| `karotter_csrf` | CSRFトークン |

### LocalStorage
| キー | 説明 |
|------|------|
| `karotter:accounts` | マルチアカウント一覧 |
| `karotter:active-account-id` | アクティブアカウントID |
| `deviceId` | デバイス識別子 |
| `theme` | テーマ (light/dark) |
| `locale` | 言語 (ja/en) |
| `appearance-settings-v1` | 外観設定 (fontScale, uiScale等) |
| `karotter-call-settings` | 通話設定 |
| `karotter:post-drafts:${userId}` | 下書き |
| `push-notifications-enabled` | プッシュ通知ON/OFF |

---

## フロントエンドルート

### メイン
| パス | ページ |
|------|--------|
| `/` | ホーム/タイムライン |
| `/login` | ログイン |
| `/register` | 登録 |
| `/search` | 検索 |
| `/notifications` | 通知 |
| `/notifications/grouped-posts` | グループ化通知 |
| `/follow-requests` | フォローリクエスト |
| `/bookmarks` | ブックマーク |
| `/settings` | 設定 |
| `/profile/:username` | ユーザープロフィール |
| `/posts/:id` | 投稿詳細 |
| `/:username/status/:id` | 投稿パーマリンク |
| `/dm` | DMグループ一覧 |
| `/dm/:groupId` | DMチャット |
| `/social` | サークル/リスト |
| `/draw-chat` | 絵チャロビー |
| `/draw-chat/rooms/:roomId` | 絵チャルーム |
| `/spaces` | スペース一覧 |
| `/spaces/:spaceId` | スペース詳細 |
| `/report` | 通報 |
| `/contact` | お問い合わせ |

### 法的ページ
`/terms`, `/privacy`, `/cookie-policy`, `/ai-policy`, `/enforcement-policy`, `/child-safety-policy`, `/api-bot-terms`, `/creator-monetization-terms`, `/repeat-infringer-policy`, `/account-memorial-inheritance-policy`, `/business-transfer-data-succession-policy`

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

## APIドメイン

| ドメイン | 状態 | HSTS |
|---------|------|------|
| `api.karotter.com` | Active (プライマリ) | max-age=0 (無効) |
| `api.karotter.jp` | Active (ミラー) | max-age=31536000 |
| `apikarotter.karon.jp` | Active (ミラー) | max-age=15552000 |
| `api.karotter.net` | DNS解決不可 | - |

- 3ドメインは同一バックエンド。トークンはクロスドメインで有効
- Socket.IO も全ドメインで利用可能

### CORS許可オリジン
`karotter.com`, `karotter.jp`, `karotter.net`, `karotter.karon.jp`, `apikarotter.karon.jp`, `http://localhost:5173` (dev)

---

## 注意事項

- `/api/users/me` は「me」というユーザー名を検索する (現在のユーザーは `/api/auth/me`)
- 同様に `/api/users/settings`, `/api/users/notifications` 等もユーザー名として解釈される
- 画像アップロードのフィールド名: 投稿は `media`、DMは `attachments` (間違えると500)
- 画像と動画の同時投稿不可 (400)
- Cookie認証 (`karotter_at`, `karotter_rt`) が一部エンドポイントで必要。`requests.Session()` で自動処理

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
