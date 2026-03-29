# 通知グルーピング構造

通知はグルーピングされて返される。

---

## グルーピングフィールド

| フィールド | 説明 |
|-----------|------|
| `groupKey` | グループキー |
| `actors[]` | アクションしたユーザー配列 |
| `actorCount` | アクターの総数 |
| `posts[]` | 関連投稿配列 |
| `postCount` | 投稿の総数 |
| `notificationIds[]` | 通知ID配列 |

---

## 通知タイプ一覧

| type | 説明 |
|------|------|
| `REPLY` | 返信 |
| `MENTION` | メンション |
| `FOLLOW` | フォロー |
| `FOLLOW_REQUEST` | フォローリクエスト |
| `LIKE` | いいね |
| `REKAROT` | リカロート |
| `QUOTE` | 引用 |
| `DM` | ダイレクトメッセージ |

---

## 通知オブジェクト例

```json
{
  "id": "...",
  "type": "REPLY",
  "actor": { ... },
  "actors": [{ ... }],
  "post": { ... },
  "posts": [{ ... }],
  "createdAt": "..."
}
```

グループ化された通知 (`GET /notifications/grouped-posts`) では、同じ投稿に対する複数の通知がまとめられ、`actors` 配列に全アクターが含まれる。
