# Developer API エンドポイント

APIキーを使用する第三者アプリ向けエンドポイント。セルフBot APIのサブセット。

---

## 概要

**Base URL**: `https://api.karotter.com/api/developer`

**認証**: `Authorization: Bearer {apiKey}` (APIキーは `kar_live_` プレフィックス)

---

## 利用可能エンドポイント

| メソッド | パス | 説明 |
|---------|------|------|
| GET | `/posts` | 自分の投稿一覧 |
| GET | `/posts/{id}` | 投稿詳細 |
| GET | `/posts/{id}/replies` | 返信一覧 |
| POST | `/posts` | 投稿作成 |
| GET | `/timeline` | タイムライン |
| GET | `/search?q={keyword}` | 投稿検索 |
| GET | `/bookmarks` | ブックマーク一覧 |
| GET | `/notifications` | 通知一覧 |
| GET | `/users/{id}` | ユーザー情報 |
| POST | `/posts/{id}/like` | いいね |
| DELETE | `/posts/{id}/like` | いいね取消 |
| POST | `/posts/{id}/bookmark` | ブックマーク追加 |
| POST | `/posts/{id}/rekarot` | リカロート |
| DELETE | `/posts/{id}/rekarot` | リカロート取消 |

---

## Developer APIで利用不可

以下はセルフBot API (`/api/`) でのみ利用可能:

- `DELETE /posts/{id}` (投稿削除)
- `/auth/*` (認証関連)
- `/dm/*` (ダイレクトメッセージ)
- `/follow/*` (フォロー操作)
- `/draw/*` (絵チャ)
- `/radio/*` (ラジオ)

---

## 備考

- APIキーは `kar_live_` プレフィックスの形式
- `/apikeys` エンドポイントでAPIキーを管理可能 (GET/POST/DELETE)
- Developer APIは認証ミドルウェアがルートマッチング前に実行されるため、無効なキーでは全パスが401を返す
- 認証系は現在リライト中の可能性あり
