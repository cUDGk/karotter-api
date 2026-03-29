# 通知 (Notifications) エンドポイント

通知一覧、未読数、既読、削除、プッシュ通知関連の全エンドポイント。

---

## 通知一覧

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

---

## 未読通知数

```
GET /notifications/unread/count

Response 200: {"count": 0}
```

---

## グループ化された投稿通知

```
GET /notifications/grouped-posts?limit=15

Response 200: {"posts": [...]}
```

---

## 全通知既読

```
PATCH /notifications/read-all

Response 200: {"message": "すべての通知を既読にしました"}
```

---

## 通知削除

```
DELETE /notifications/{id}        → {"message": "通知を削除しました"}
DELETE /notifications/all         → 500 (サーバーバグ、未修正)
```

---

## プッシュ通知登録

```
POST /notifications/push/register     Body: {"token": "fcm-token", "deviceId": "uuid"}
POST /notifications/push/unregister   Body: {"token": "fcm-token"}
```
