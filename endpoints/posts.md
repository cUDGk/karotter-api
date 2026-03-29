# 投稿 (Posts) エンドポイント

投稿の作成・取得・編集・削除、いいね、リアクション、投票、リカロート、ブックマーク関連の全エンドポイント。

---

## カロート投稿

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

**制限事項**:
- 画像と動画の同時投稿不可 (`400: 画像と動画は同時に送信できません`)
- フィールド名は `media` (投稿) / `attachments` (DM) -- 間違えると500エラー

---

## 投稿取得

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

---

## 投稿編集

```
PUT /posts/{id}
Content-Type: multipart/form-data

Fields: content (必須), media, mediaAlts, etc.
Error: 400 {"error": "本文を空にすることはできません"}
```
- 自分の投稿のみ編集可能 (403)

---

## 投稿削除

```
DELETE /posts/{id}

Response 200: {"message": "カロートを削除しました"}
```
- 他人の投稿: `403 {"error": "このカロートを削除する権限がありません"}`
- 削除後のGET: `404 {"error": "カロートが見つかりません"}`

---

## 投稿の返信一覧

```
GET /posts/{id}/replies

Response 200: {"replies": [...]}  ← 投稿オブジェクト配列
```

---

## いいねしたユーザー一覧

```
GET /posts/{id}/likes

Response 200: {"users": [...]}  ← ユーザーオブジェクト配列
```

---

## 引用一覧

```
GET /posts/{id}/quotes

Response 200: {"quotes": [...]}  ← 引用投稿オブジェクト配列
```

---

## リカロートしたユーザー一覧

```
GET /posts/{id}/rekarots

Response 200: {"users": [...]}  ← ユーザーオブジェクト配列
```

---

## 会話スレッド

```
GET /posts/{id}/conversation
  Response 200: {"rootPostId", "isParticipant", "hasLeftConversation", "participants": [user...]}

POST /posts/{id}/conversation/leave  → 会話から離脱
```

---

## 投稿アナリティクス

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

---

## タイムライン

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

---

## トレンド

```
GET /posts/trending
```
- 認証不要

---

## おすすめ投稿

```
GET /posts/recommended
```

---

## 予約投稿

```
GET    /posts/scheduled/me       → {"scheduledPosts": [...]}
DELETE /posts/scheduled/{id}     → 予約投稿削除
```

---

## 自分の返信

```
GET /posts/me/replies?limit=15

Response 200: {"replies": [...]}
```

---

## 閲覧数バッチ送信

```
POST /posts/batch-views
Body: {"postIds": [12345, 67890]}

Response 200: {"recorded": 2}
```

---

## ベータUIフィードバック

```
POST /posts/feedback/beta-survey
Content-Type: application/json

{
  "preference": "beta" | "current"
}
```

---

## 返信先ユーザー取得

```
GET /posts/{id}/reply-targets

Response 200: {"rootPostId", "selectedUserIds": [...], "excludedUserIds": [...], "candidates": [user...]}
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
