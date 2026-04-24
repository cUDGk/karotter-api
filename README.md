<div align="center">

# Karotter API Reference

### Karotter (カロッター) の非公式 API リファレンス

[![Status](https://img.shields.io/badge/Status-非公式_·_都度更新-red?style=flat)](#注意事項)
[![Endpoints](https://img.shields.io/badge/Endpoints-14_カテゴリ-blue?style=flat)](#ファイル構成)
[![Auth](https://img.shields.io/badge/認証-APIキー_·_Cookie-green?style=flat)](#認証方式)
[![Realtime](https://img.shields.io/badge/リアルタイム-Socket.IO-purple?style=flat)](endpoints/socket.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat)](LICENSE)

**フロントエンド JS バンドル解析 + 実 API テスト + セキュリティ診断レポートから作成した都度更新ドキュメント。**

---

</div>

> **⚠️ 注意**: 恐らく仕様変更が入っていますが対応できていません。単に Bot を作りたいだけなら [karotter-selfbot](https://github.com/cUDGk/karotter-selfbot) を検討してください。

## 概要

Karotter (カロッター) の非公式 API リファレンス。フロントエンドの JS バンドル解析、実際の API テスト、セキュリティ診断レポート、および公式ドキュメントに基づいて作成しています。


## 特徴

| 項目 | 内容 |
|---|---|
| エンドポイント | 14 カテゴリ、約 145 件 |
| 認証 | API キー / Cookie |
| リアルタイム | Socket.IO イベント ~35 種 |
| 作成元 | JS バンドル解析 + 実 API テスト + セキュリティ診断 |
| 対象 | `karotter.com` / `karotter.jp` |


## Base URL

| 用途 | Base URL |
|------|----------|
| セルフBot (自アカウント操作) | `https://api.karotter.com/api/` |
| 開発者向け (第三者アプリ) | `https://api.karotter.com/api/developer` |

> `https://karotter.com/api` でもアクセス可能（Cloudflare経由）

---

## 共通ヘッダー

全リクエストに以下のヘッダーを付与する。

| ヘッダー | 値 | 必須 | 備考 |
|---------|-----|------|------|
| `X-Client-Type` | `web` | 推奨 | `web` 以外でも動作するがアカウント凍結リスクあり |
| `X-Device-Id` | UUID v4 | 推奨 | セッション識別用、固定値でOK |
| `X-CSRF-Token` | CSRFトークン | 書き込み系 | POST/PUT/DELETE で必要 |

### 認証方式

**方式1: APIキー認証** (開発者向け)
```
Authorization: Bearer {apiKey}
```
- `/apikeys` エンドポイントで発行可能

**方式2: Cookie認証** (セルフBot)
```
Cookie: {ログイン時に返却されるCookie}
```
- ログインレスポンスの `Set-Cookie` をそのまま使用

---

## レート制限

レスポンスヘッダー: `ratelimit-policy`, `ratelimit-limit`, `ratelimit-remaining`, `ratelimit-reset`

| エンドポイント | 制限 |
|--------------|------|
| POST /auth/login | 5 req / 60s |
| POST /follow/{id} | 5 req / 60s |
| POST /contact | 5 req / 3600s |
| POST /reports | 10 req / 900s |
| PATCH /users/profile | 15 req / 60s |
| POST /posts/{id}/like, /react, /bookmark, /rekarot | 30 req / 60s |
| POST /posts | 50 req / 3600s |
| その他全般 | 100 req / 60s |

---

## HTTPステータスコード

| コード | 意味 |
|-------|------|
| 200 | 成功 |
| 201 | 作成成功 (投稿、メッセージ等) |
| 400 | バリデーションエラー |
| 401 | 未認証 / トークン無効 |
| 403 | 権限なし |
| 404 | リソースが見つからない |
| 409 | 競合 (重複操作等) |
| 429 | レート制限超過 |
| 500 | サーバー内部エラー |

---

## エラーレスポンス形式

```json
// バリデーション・ビジネスエラー
{"error": "メッセージ", "status": 400}

// 認証エラー
{"error": "無効なトークンです"}

// 権限エラー
{"error": "このカロートを削除する権限がありません", "status": 403}

// 管理者権限
{"error": "管理者権限が必要です", "status": 403}
```

---

## ファイル構成

| ファイル | 説明 |
|---------|------|
| [endpoints/auth.md](endpoints/auth.md) | 認証 (ログイン、登録、セッション管理等) |
| [endpoints/posts.md](endpoints/posts.md) | 投稿 (作成、取得、編集、削除、いいね、リアクション、投票、リカロート、ブックマーク) |
| [endpoints/users.md](endpoints/users.md) | ユーザー (プロフィール、設定、アカウント管理) |
| [endpoints/follow.md](endpoints/follow.md) | フォロー (フォロー、ブロック、ミュート) |
| [endpoints/notifications.md](endpoints/notifications.md) | 通知 |
| [endpoints/dm.md](endpoints/dm.md) | ダイレクトメッセージ |
| [endpoints/search.md](endpoints/search.md) | 検索 (統合、投稿、ユーザー、ハッシュタグ、トレンド、ディスカバー) |
| [endpoints/social.md](endpoints/social.md) | ソーシャル (サークル、リスト、ストーリー、リンクプレビュー) |
| [endpoints/draw.md](endpoints/draw.md) | Draw-chat (絵チャ) |
| [endpoints/radio.md](endpoints/radio.md) | ラジオ / スペース |
| [endpoints/admin.md](endpoints/admin.md) | 管理パネル |
| [endpoints/developer.md](endpoints/developer.md) | Developer API |
| [endpoints/other.md](endpoints/other.md) | 規約、通報、お問い合わせ、APIキー、埋め込み |
| [endpoints/socket.md](endpoints/socket.md) | Socket.IO リアルタイム通信 |
| [responses/post.md](responses/post.md) | 投稿レスポンス構造 |
| [responses/notification.md](responses/notification.md) | 通知グルーピング構造 |
| [examples/python.md](examples/python.md) | Python Bot 実装例 |

---

## APIドメイン

| ドメイン | 状態 | HSTS |
|---------|------|------|
| `api.karotter.com` | Active (プライマリ) | max-age=0 (無効) |
| `api.karotter.jp` | Active (ミラー) | max-age=31536000 |
| `apikarotter.karon.jp` | Active (ミラー) | max-age=15552000 |
| `api.karotter.net` | DNS解決不可 | - |

- 3ドメインは同一バックエンド。トークンはクロスドメインで有効
- Socket.IO も全ドメインで利用可能

### CORS許可オリジン

`karotter.com`, `karotter.jp`, `karotter.net`, `karotter.karon.jp`, `apikarotter.karon.jp`, `http://localhost:5173` (dev)

---

## 技術スタック

| 項目 | 詳細 |
|------|------|
| Backend | Node.js (Express) + Prisma |
| Database | PostgreSQL |
| Frontend | Vite + React |
| CDN/WAF | Cloudflare |
| リアルタイム | Socket.IO (WebSocketのみ) |
| 認証 | JWT (HS256) + CSRF |
| SSL | Let's Encrypt |

---

## Cookie / LocalStorage

### Cookie
| 名前 | 説明 |
|------|------|
| `karotter_at` | アクセストークン |
| `karotter_rt` | リフレッシュトークン |
| `karotter_csrf` | CSRFトークン |

### LocalStorage
| キー | 説明 |
|------|------|
| `karotter:accounts` | マルチアカウント一覧 |
| `karotter:active-account-id` | アクティブアカウントID |
| `deviceId` | デバイス識別子 |
| `theme` | テーマ (light/dark) |
| `locale` | 言語 (ja/en) |
| `appearance-settings-v1` | 外観設定 (fontScale, uiScale等) |
| `karotter-call-settings` | 通話設定 |
| `karotter:post-drafts:${userId}` | 下書き |
| `push-notifications-enabled` | プッシュ通知ON/OFF |

---

## フロントエンドルート

### メイン
| パス | ページ |
|------|--------|
| `/` | ホーム/タイムライン |
| `/login` | ログイン |
| `/register` | 登録 |
| `/search` | 検索 |
| `/notifications` | 通知 |
| `/notifications/grouped-posts` | グループ化通知 |
| `/follow-requests` | フォローリクエスト |
| `/bookmarks` | ブックマーク |
| `/settings` | 設定 |
| `/profile/:username` | ユーザープロフィール |
| `/posts/:id` | 投稿詳細 |
| `/:username/status/:id` | 投稿パーマリンク |
| `/dm` | DMグループ一覧 |
| `/dm/:groupId` | DMチャット |
| `/social` | サークル/リスト |
| `/draw-chat` | 絵チャロビー |
| `/draw-chat/rooms/:roomId` | 絵チャルーム |
| `/spaces` | スペース一覧 |
| `/spaces/:spaceId` | スペース詳細 |
| `/report` | 通報 |
| `/contact` | お問い合わせ |

### 法的ページ
`/terms`, `/privacy`, `/cookie-policy`, `/ai-policy`, `/enforcement-policy`, `/child-safety-policy`, `/api-bot-terms`, `/creator-monetization-terms`, `/repeat-infringer-policy`, `/account-memorial-inheritance-policy`, `/business-transfer-data-succession-policy`

---

## 注意事項

- `/api/users/me` は「me」というユーザー名を検索する (現在のユーザーは `/api/auth/me`)
- 同様に `/api/users/settings`, `/api/users/notifications` 等もユーザー名として解釈される
- 画像アップロードのフィールド名: 投稿は `media`、DMは `attachments` (間違えると500)
- 画像と動画の同時投稿不可 (400)
- Cookie認証 (`karotter_at`, `karotter_rt`) が一部エンドポイントで必要。`requests.Session()` で自動処理

---

## メディア

### メディアURLの構造
```
画像: /uploads/posts/{uuid}.{ext}
DM添付: /uploads/dm/{uuid}.{ext}
アバター: /uploads/avatars/avatar_{userId}_{timestamp}.webp
ヘッダー: /uploads/headers/header_{userId}_{timestamp}.webp

フルURL: https://karotter.com{path}
```

### メディアフィールド (投稿)
```json
{
  "mediaUrls": ["/uploads/posts/uuid.png"],
  "mediaTypes": ["image"],
  "mediaAlts": [""],
  "mediaSpoilerFlags": [false],
  "mediaR18Flags": [false]
}
```
- `mediaTypes`: `"image"` / `"video"`

### メディアフィールド (DM)
```json
{
  "attachmentUrls": ["/uploads/dm/uuid.png"],
  "attachmentTypes": ["image/png"],
  "attachmentAlts": [""],
  "attachmentSpoilerFlags": [false],
  "attachmentR18Flags": [false]
}
```
- フィールド名が投稿とは異なる (`media*` vs `attachment*`)
- `attachmentTypes` は MIME type (`image/png` 等)

---

## クエリパラメータ

多くのエンドポイントで以下のパラメータが使用可能。

### ページネーション

**Offset方式** (従来):
```
?page=1&limit=15
```

**Cursor方式** (推奨、無限スクロール用):
```
?limit=12&cursor={lastId}
```
1. 初回は `cursor` なしでリクエスト → 最新N件を取得
2. レスポンスの最後の投稿IDを `cursor` に指定 → それ以前のデータを取得
3. データの重複なく過去を遡れる

## ライセンス

MIT License — 詳細は [LICENSE](LICENSE) を参照。
