# DM (ダイレクトメッセージ) エンドポイント

DMグループ、メッセージ送受信、既読、グループ操作、通話関連の全エンドポイント。

---

## DMグループ一覧 / 作成

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

---

## メッセージ取得

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

---

## メッセージ送信

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

---

## 既読マーク

```
POST /dm/groups/{groupId}/read
```

---

## 1対1 DM開始

```
POST /dm/start
Content-Type: application/json

{
  "targetUserId": 12345
}
```

---

## グループ操作

```
POST   /dm/groups/{groupId}/clear              → チャット履歴クリア (自分の表示のみ削除)
POST   /dm/groups/{groupId}/leave              → グループ退出 (1対1DMは退出不可、クリアを使用)
POST   /dm/groups/{groupId}/members            → メンバー追加 (メール認証必須)
DELETE /dm/groups/{groupId}/members/{userId}    → メンバー削除
POST   /dm/groups/{groupId}/request/accept     → DMリクエスト承認
POST   /dm/groups/{groupId}/request/reject     → DMリクエスト拒否
```

---

## メッセージ操作

```
DELETE /dm/messages/{id}              → メッセージ削除
PATCH  /dm/messages/{id}              → メッセージ編集
POST   /dm/messages/{id}/reactions    → メッセージリアクション
POST   /dm/messages/{id}/poll/vote    → DM内投票
```

---

## 通話

```
GET  /dm/groups/{groupId}/call          → 通話状態確認
POST /dm/groups/{groupId}/call/start    → 通話開始
POST /dm/groups/{groupId}/call/join     → 通話参加
POST /dm/groups/{groupId}/call/leave    → 通話退出
```
