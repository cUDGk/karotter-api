# ユーザー (Users) エンドポイント

ユーザー情報取得、プロフィール編集、設定、アカウント管理関連の全エンドポイント。

---

## ユーザー情報取得

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

---

## ユーザーのサブリソース

```
GET /users/{userId}/posts?limit=15         → {"posts": [...]}
GET /users/{userId}/likes?limit=15         → {"posts": [...]}
GET /users/{userId}/media?limit=15         → {"posts": [...]} (メディア付き投稿のみ)
GET /users/{userId}/replies?limit=15       → {"posts": [...]}
GET /users/{userId}/mutual-followers       → {"users": [...], "pagination": {"hasNext", "nextCursor"}}
```

---

## フォロワー / フォロー中一覧

```
GET /users/{userId}/followers?limit=100
GET /users/{userId}/following?limit=1000

Response 200: {"users": [...], "pagination": {"hasNext", "nextCursor"}}
```
- ユーザーオブジェクト: id, username, displayName, avatarUrl, bio, followersCount, isPrivate, officialMark, is_following, follow_request_sent

---

## おすすめユーザー

```
GET /users/recommended?limit=3

Response 200: {"users": [...]}
```

---

## ユーザー名クォータ確認

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

---

## プロフィール更新

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

---

## ステータス更新

```
PATCH /users/status
Body: {"status": "ONLINE"}  ← ONLINE / OFFLINE / INVISIBLE
      {"statusMessage": "作業中"}  ← ステータスメッセージ（任意）

Response 200: {"message": "ステータスを更新しました", "status": "ONLINE", "statusMessage": "..."}
```

---

## パスワード変更

```
PATCH /users/password
Body: {"currentPassword": "old", "newPassword": "new"}

Error: 400 {"error": "現在のパスワードが正しくありません"}
```

---

## ユーザー名変更

```
PATCH /users/username
Body: {"username": "newname"}

Response 200: {"message": "ユーザー名を変更しました", "user": {...}, "limit": {"windowDays": 14, "maxChanges": 5}}
変更なし: {"message": "ユーザー名に変更はありません"}
```
- レート制限: 14日間で最大5回

---

## ピン留め投稿変更

```
PATCH /users/profile/pinned-post
Body: {"postId": 12345}  ← ピン留め
Body: {}                 ← ピン解除

Response 200: {"message": "固定カロートを解除しました", "pinnedPostId": null}
```

---

## ユーザー設定

```
PATCH /users/settings
Body: {} ← 変更したいフィールドのみ

Response 200: 全設定を返す
```

設定フィールド一覧:
- `isPrivate`: 非公開アカウント
- `onlineStatusVisibility`: オンライン状態の公開範囲
- `showLikedPosts`: いいね投稿を表示
- `showReadReceipts`: 既読表示
- `directMessagesEnabled`: DM有効
- `mutedKeywords`: ミュートキーワード配列
- `dmRequestPolicy`: DMリクエストポリシー
- `notifyLikes`, `notifyRekarots`, `notifyReplies`, `notifyMentions`, `notifyFollows`, `notifyQuotes`, `notifyReactions`, `notifyDMs`: 通知設定 (各boolean)
- `notificationMuteNonFollowing`, `notificationMuteNonFollowers`, `notificationMuteNewAccounts`, `notificationMuteNoAvatar`: 通知ミュート設定
- `showHiddenPosts`, `showParodyAccounts`, `showBotAccounts`, `showR18Content`: 表示設定
- `isBotAccount`, `isParodyAccount`, `hideProfileFromMinors`: アカウント属性

---

## アカウント削除

```
DELETE /users/account
Body: {"password": "current_password"}

Error: 401 {"error": "パスワードが正しくありません"}
```
- 正しいパスワード必須。取り消し不可。

---

## アバター / ヘッダー更新

```
POST /profile/avatar    → アバター (multipart/form-data)
POST /profile/header    → ヘッダー画像 (multipart/form-data)
```
