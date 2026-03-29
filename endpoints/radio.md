# ラジオ (Radio) / スペース エンドポイント

ラジオの作成・参加・退出・終了、メッセージ、設定、参加者管理関連の全エンドポイント。

---

## ラジオ作成

```
POST /radio
Content-Type: application/json

{
  "title": "ラジオタイトル"
}
```
- タイトルは1〜100文字

---

## アクティブなラジオ一覧

```
GET /radio/active
```

---

## ラジオ詳細

```
GET /radio/{id}
```

---

## ラジオ内メッセージ

```
GET  /radio/{id}/messages    → メッセージ一覧
POST /radio/{id}/messages    → メッセージ送信
```

---

## ラジオ設定

```
PATCH /radio/{id}/settings
```

---

## 参加者管理

```
PATCH  /radio/{id}/participants/{userId}/role            → ロール変更
PATCH  /radio/{id}/participants/{userId}/mute            → ミュート
POST   /radio/{id}/participants/{userId}/invite-speaker  → スピーカー招待
DELETE /radio/{id}/participants/{userId}/invite-speaker  → スピーカー招待取消
POST   /radio/{id}/accept-speaker-invite                → スピーカー招待承諾
POST   /radio/{id}/request-speaker                      → スピーカーリクエスト
```

---

## ラジオ参加 / 退出 / 終了

```
POST /radio/{id}/join
POST /radio/{id}/leave
POST /radio/{id}/end
```
