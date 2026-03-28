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
POST /auth/resend-verification    → メール認証再送 (600秒クールダウン)
POST /auth/me/email               → メールアドレス変更
```

### セッション管理
```
GET  /auth/sessions                  → 全セッション一覧
DELETE /auth/sessions/:id            → 特定セッション削除
DELETE /auth/sessions/others         → 他の全セッションを削除
DELETE /auth/sessions/all            → 全セッション削除
POST /auth/switch-session            → セッション切り替え
  Body: {"sessionId": "..."} または {"userId": ...}
  ※ セッションIDまたはユーザーIDが必要
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

### 会話スレッドから離脱
```
POST /posts/{id}/conversation/leave
```

### 投稿アナリティクス
```
GET /posts/{id}/analytics
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
POST   /posts/{id}/rekarot    → リカロート
DELETE /posts/{id}/rekarot    → リカロート取消
```

---

## ブックマーク

### ブックマーク一覧
```
GET /posts/me/bookmarks
Authorization: Bearer {token}
```

### ブックマーク追加 / 解除
```
POST   /posts/{id}/bookmark    → ブックマーク追加
DELETE /posts/{id}/bookmark    → ブックマーク解除
```

---

## DM (ダイレクトメッセージ)

### DMグループ一覧 / 作成
```
POST /dm/groups                → グループ作成 (Body: {"userIds": [userId, ...]})
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
  "userId": 12345
}
```

### グループ操作
```
POST   /dm/groups/{groupId}/clear              → チャット履歴クリア
POST   /dm/groups/{groupId}/leave              → グループ退出
POST   /dm/groups/{groupId}/members            → メンバー追加
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

## 規約

```
GET /legal/summary    → 規約バージョン・発効日
GET /legal/terms      → 利用規約全文
GET /legal/privacy    → プライバシーポリシー全文
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

秘匿パス: `/control-room-x9k2`
全エンドポイントは管理者権限が必要 (`403: 管理者権限が必要です`)

### ダッシュボード / 統計
```
GET /control-room-x9k2/dashboard
GET /control-room-x9k2/analytics
GET /control-room-x9k2/stats
GET /control-room-x9k2/stats/users
GET /control-room-x9k2/stats/posts
GET /control-room-x9k2/stats/daily
```

### ユーザー管理
```
GET    /control-room-x9k2/users
GET    /control-room-x9k2/users/{id}
POST   /control-room-x9k2/users
PATCH  /control-room-x9k2/users/{id}/ban
PATCH  /control-room-x9k2/users/{id}/account
PATCH  /control-room-x9k2/users/{id}/official-mark
PATCH  /control-room-x9k2/users/{id}/flags
DELETE /control-room-x9k2/users/{id}
```

### 投稿管理
```
GET    /control-room-x9k2/posts
GET    /control-room-x9k2/posts/{id}
PATCH  /control-room-x9k2/posts/{id}/flags
DELETE /control-room-x9k2/posts/{id}
```

### ストーリー管理
```
GET    /control-room-x9k2/stories
PATCH  /control-room-x9k2/stories/{id}/flags
DELETE /control-room-x9k2/stories/{id}
```

### 通報 / モデレーション
```
GET  /control-room-x9k2/reports
GET  /control-room-x9k2/reports/pending
POST /control-room-x9k2/reports/resolve
GET  /control-room-x9k2/moderation
```

### BAN
```
GET  /control-room-x9k2/bans
GET  /control-room-x9k2/bans/{id}
POST /control-room-x9k2/bans
GET  /control-room-x9k2/ip-bans
```

### コンテンツ管理
```
GET /control-room-x9k2/badges
GET /control-room-x9k2/emoji
GET /control-room-x9k2/frames         → アバターフレーム
GET /control-room-x9k2/gacha
GET /control-room-x9k2/themes
GET /control-room-x9k2/announcements
POST /control-room-x9k2/announcements
```

### 申請 / リクエスト
```
GET /control-room-x9k2/verification-requests
GET /control-room-x9k2/bot-requests
GET /control-room-x9k2/appeals
```

### システム
```
GET  /control-room-x9k2/settings
POST /control-room-x9k2/settings
GET  /control-room-x9k2/config
GET  /control-room-x9k2/system
GET  /control-room-x9k2/logs
GET  /control-room-x9k2/audit
GET  /control-room-x9k2/maintenance
POST /control-room-x9k2/maintenance
GET  /control-room-x9k2/backup
POST /control-room-x9k2/backup
GET  /control-room-x9k2/cron
GET  /control-room-x9k2/webhooks
GET  /control-room-x9k2/api-keys
GET  /control-room-x9k2/sessions
GET  /control-room-x9k2/queue
GET  /control-room-x9k2/tasks
```

### その他管理
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
GET /control-room-x9k2/roles
GET /control-room-x9k2/permissions
GET /control-room-x9k2/radio
GET /control-room-x9k2/draw
GET /control-room-x9k2/dm
GET /control-room-x9k2/emails
GET /control-room-x9k2/actions
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
| `draw:room-state` | S→C | 絵チャルーム状態 |

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
