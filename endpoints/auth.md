# 認証 (Auth) エンドポイント

認証関連の全エンドポイント (`/auth/*`)。

---

## ログイン

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
  "message": "ログインに成功しました",
  "accessToken": "eyJ...",
  "sessionId": "uuid",
  "deviceId": "uuid",
  "user": {
    "id": 15459,
    "username": "claude",
    "displayName": "claude",
    "email": "",
    "avatarUrl": "/uploads/avatars/...",
    "emailVerified": false
  }
}
```
- `accessToken`: JWT (HS256), 30分有効
- Set-Cookie: `karotter_at` (HttpOnly, Secure, SameSite=Strict, 1h), `karotter_rt` (HttpOnly, Secure, 30d), `karotter_csrf` (Secure, 30d)
- レート制限: 5 req / 60s

---

## CSRFトークン

```
GET    /auth/csrf-token     → トークン取得
DELETE /auth/csrf-token     → トークン無効化

Response 200:
{
  "csrfToken": "f7eb0b1a-..."
}
```
- 書き込み系リクエスト (POST/PUT/DELETE) に `X-CSRF-Token: {csrfToken}` が必要

---

## トークンリフレッシュ

```
POST /auth/refresh-token
Cookie: karotter_rt={refreshToken}
```

---

## ログアウト

```
POST /auth/logout

Response 200: {"message": "ログアウトしました"}
```
- 3つのCookieをクリア (karotter_at, karotter_rt, karotter_csrf)

---

## 自分の情報取得

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

---

## アカウント登録

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

---

## パスワードリセット

```
POST /auth/forgot-password
  Body: {"email": "user@example.com"}
  Response 200: {"message": "パスワードリセットメールを送信しました"}
  ※ 存在しないメールでも200を返す（ユーザー列挙防止）

POST /auth/reset-password
  Body: {"token": "reset-token", "password": "newPassword"}
  Error: 400 {"error": "無効なトークン形式です"}
```

---

## メール認証

```
POST /auth/verify-email
  Body: {"token": "verify-token"}
  Error: 400 {"error": "無効または期限切れのトークンです。設定画面から確認メールを再送信してください。"}

POST /auth/me/email/resend
  Error: 400 {"error": "先にメールアドレスを登録してください"}

POST /auth/resend-verification
  Body: {"email": "user@example.com"}
  Error: 400 {"error": "有効なメールアドレスを入力してください"}

POST /auth/me/email
  Body: {"email": "new@example.com"}
  Response 200: {"message": "確認メールを送信しました", "email": "new@example.com", "cooldownSeconds": 600}
```

---

## セッション管理

```
GET  /auth/sessions                  → 全セッション一覧
  Response: セッション配列。各セッション: id, deviceId, clientType, deviceName, userAgent, createdAt, lastUsedAt, expiresAt, isCurrent
DELETE /auth/sessions/:id            → 特定セッション削除
DELETE /auth/sessions/others         → 他の全セッションを削除
  Response 200: {"message": "他のセッションを失効しました", "revokedCount": 5}
DELETE /auth/sessions/all            → 全セッション削除
POST /auth/switch-session            → セッション切り替え (マルチアカウント用)
  Body: {"sessionId": "uuid"} または {"userId": 12345}
  Response 200: {"accessToken": "jwt...", "sessionId": "uuid", "user": {...}}
  ※ 新しいアクセストークンとセッションIDが発行される
  ※ 空Body: 400 {"error": "セッションIDまたはユーザーIDが必要です"}
  ※ 無効なセッション: 401 {"error": "無効なセッションです"}
POST /auth/session-unread-snapshots  → 未読スナップショット
  Body: {"deviceId": "uuid"}  ※ deviceId必須
  Response 200: {"snapshots": [...]}
```
