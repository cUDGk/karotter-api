# フォロー (Follow) エンドポイント

フォロー、フォロー解除、フォロワー削除、フォローリクエスト、ブロック、ミュート関連の全エンドポイント。

---

## フォローする

```
POST /follow/{userId}

Response 200: {"message": "フォローしました"}
```
- レート制限: 5 req / 60s (厳しい)
- 重複: `400 {"error": "既にフォローしています"}`
- 自分自身: `400`

---

## フォロー解除

```
DELETE /follow/{userId}

Response 200: {"message": "フォローを解除しました"}
```

---

## フォロワー削除

```
DELETE /follow/follower/{userId}    → フォロワーを外す
  Error: 400 {"error": "このユーザーはフォロワーではありません"}
```

---

## フォローリクエスト

```
GET  /follow/requests/pending          → 保留中の受信リクエスト
POST /follow/requests/{id}/accept      → 承認
POST /follow/requests/{id}/reject      → 拒否
  Error: 404 {"error": "リクエストが見つかりません"}
```

---

## ブロック

```
GET    /follow/block              → ブロック一覧 {"users": [...]}
POST   /follow/block/{userId}     → ブロック追加
DELETE /follow/block/{userId}     → ブロック解除
  Error: 400 {"error": "ブロックしていません"}
```

---

## ミュート

```
GET    /follow/mute               → ミュート一覧 {"users": [...]}
POST   /follow/mute/{userId}      → ミュート追加
DELETE /follow/mute/{userId}      → ミュート解除
  Error: 400 {"error": "ミュートしていません"}
```
