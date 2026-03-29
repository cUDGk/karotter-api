# ソーシャル (Social) エンドポイント

サークル、リスト、ストーリー、リンクプレビュー関連の全エンドポイント。

---

## サークル

```
GET    /social/circles                        → 一覧
POST   /social/circles                        → 作成
DELETE /social/circles/{id}                   → 削除
POST   /social/circles/{id}/members           → メンバー追加
DELETE /social/circles/{id}/members/{userId}  → メンバー削除
```

---

## リスト

```
GET    /social/lists                          → 一覧
POST   /social/lists                          → 作成
DELETE /social/lists/{id}                     → 削除
POST   /social/lists/{id}/members             → メンバー追加
DELETE /social/lists/{id}/members/{userId}    → メンバー削除
GET    /social/lists/{id}/posts               → リストの投稿一覧
```

---

## ストーリー

```
GET    /social/stories                → ストーリー一覧
POST   /social/stories                → ストーリー投稿 (multipart/form-data)
DELETE /social/stories/{id}           → ストーリー削除
GET    /social/stories/user/{userId}  → ユーザーのストーリー
GET    /social/stories/{id}/viewers   → 閲覧者一覧 (owner only)
POST   /social/stories/{id}/views    → 閲覧記録 {"message": "閲覧を記録しました"}
POST   /social/stories/{id}/like     → ストーリーいいね
DELETE /social/stories/{id}/like     → ストーリーいいね取消
```
- ストーリーフィールド: id, authorId, mediaUrl, mediaType, caption, isR18, hideFromMinors, visibility, expiresAt, viewsCount, likesCount, hasViewed, liked

---

## リンクプレビュー

```
GET /social/link-preview?url={url}        → OGP情報取得
GET /social/link-preview-image?url={url}  → プレビュー画像取得 (常時500の可能性)
```
