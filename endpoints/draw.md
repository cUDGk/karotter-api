# Draw-chat (絵チャ) エンドポイント

絵チャルームの作成・参加・管理、レイヤー操作関連の全エンドポイント。

---

## ルーム一覧

```
GET /draw/rooms

Response 200: {"rooms": [...]}
```
- ルームフィールド: id, name, ownerId, ownerUsername, visibility, participantCount, participants, layers, createdAt, updatedAt
- 参加者: userId, username, displayName, avatarUrl, joinedAt
- レイヤー: id, name, order, visible, opacity, dataUrl (base64 PNG、大容量注意)

---

## ルーム作成

```
POST /draw/rooms
Content-Type: application/json

{
  "name": "ルーム名"
}
```

---

## ルーム詳細 (レイヤー含む)

```
GET /draw/rooms/{roomId}
```

---

## レイヤー更新

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

---

## ルーム参加

```
POST /draw/rooms/{roomId}/join
```

---

## ルーム削除

```
DELETE /draw/rooms/{roomId}
```
- オーナーのみ

---

## 招待リンクのローテーション

```
POST /draw/rooms/{roomId}/invite/rotate
```
