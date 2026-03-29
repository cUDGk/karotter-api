# 検索 (Search) エンドポイント

統合検索、投稿検索、ユーザー検索、ハッシュタグ検索、トレンド、ディスカバー関連の全エンドポイント。

---

## 統合検索

```
GET /search?q={keyword}

Response 200: {"users": [...], "posts": [...], "hashtags": [...], "pagination": {"page", "limit", "hasNext"}}
```
- 3種類の結果を同時に返す

---

## 投稿検索

```
GET /search/posts?q={keyword}&limit=15
GET /posts/search?q={keyword}               ← 500エラーの場合あり
```

---

## ユーザー検索

```
GET /search/users?q={keyword}&limit=15

Response 200: {"users": [...]}
```
- ユーザーフィールド: id, username, displayName, avatarUrl, officialMark, bio, followersCount, followingCount, is_following (snake_case注意)

---

## ハッシュタグ検索

```
GET /search/hashtags?q={keyword}&limit=15

Response 200: {"hashtags": [...], "pagination": {"page", "limit", "total", "pages"}}
```
- ハッシュタグフィールド: id, name, usageCount, trendScore, createdAt, updatedAt

---

## トレンドトピック

```
GET /search/trending/topics?limit=5

Response 200: {"trends": [...]}
```
- トレンドフィールド: token, label, type, postCount, authorCount, trendScore

---

## トレンドハッシュタグ

```
GET /search/trending/hashtags?limit=5

Response 200: {"hashtags": [...]}
```
- フィールド: id, name, usageCount, trendScore

---

## ディスカバー

```
GET /search/discover/latest?limit=12&cursor={lastPostId}    → 最新投稿
GET /search/discover/media?limit=12&cursor={lastPostId}     → メディア投稿
```
- `limit`: 取得件数 (デフォルト12)
- `cursor`: 取得開始位置。指定したIDより前の投稿を取得
- 初回は `cursor` なしで最新を取得、以降は最後の投稿IDを `cursor` にセットして無限スクロール
