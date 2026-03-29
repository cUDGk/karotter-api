# その他エンドポイント

規約、通報、お問い合わせ、APIキー、埋め込み関連のエンドポイント。

---

## 規約 (Legal)

```
GET /legal/summary    → 規約バージョン・発効日
GET /legal/terms      → 利用規約全文
GET /legal/privacy    → プライバシーポリシー全文
GET /legal/{slug}     → 法的文書（汎用、認証不要）
```

レスポンス例 (`/legal/summary`):
```json
{
  "version": "terms:2026-03-27|privacy:2026-03-27",
  "termsEffectiveDate": "2026-03-27",
  "privacyEffectiveDate": "2026-03-27"
}
```

---

## 通報 (Reports)

```
POST /reports
Content-Type: application/json

{
  "type": "POST",       // 通報対象種類
  "postId": 12345,      // 投稿通報時
  "reason": "理由"
}
```
- `type` が不正: `400 {"error": "通報対象の種類が不正です"}`
- レート制限: 10 req / 900s

---

## お問い合わせ (Contact)

```
POST /contact
Content-Type: application/json

{
  "category": "bug" | "feature_request" | "general_inquiry",
  "subject": "件名",
  "body": "本文"
}
```
- 認証不要
- レート制限: 5 req / 3600s

---

## APIキー

```
GET    /apikeys        → {"apiKeys": [...]}
POST   /apikeys        → APIキー作成 Body: {"name": "キー名"}
DELETE /apikeys/{id}   → {"message": "APIキーを削除しました"}
```
- APIキーフィールド: id, name, canReadPosts, canCreatePosts, canReadTimeline, canReadFollows, canWriteFollows, requestsPerMinute, lastUsedAt, isActive, createdAt, maskedKey
- POST時のみ `plainKey` (e.g. `kar_live_...`) が返る。以降は `maskedKey` のみ表示

---

## 埋め込み (Embed)

```
GET https://karotter.com/embed/{postId}     → 投稿の埋め込みHTML
GET https://karotter.com/oembed?url={url}   → oEmbed形式
```
- フロントエンドルート（`/api/` 配下ではない）
