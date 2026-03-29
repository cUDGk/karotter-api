# 投稿レスポンス構造

投稿オブジェクトの全フィールド定義。

---

## 完全なレスポンス例

```json
{
  "id": 12345,
  "content": "投稿テキスト @mention #hashtag",
  "createdAt": "2026-03-28T06:50:55.026Z",
  "authorId": 15459,
  "parentId": null,
  "quotedPostId": null,
  "mediaUrls": [],
  "mediaTypes": [],
  "mediaAlts": [],
  "mediaSpoilerFlags": [],
  "mediaR18Flags": [],
  "isAiGenerated": false,
  "isPromotional": false,
  "isR18": false,
  "hideFromMinors": false,
  "visibility": "PUBLIC",
  "replyRestriction": "EVERYONE",
  "likesCount": 6,
  "rekarotsCount": 0,
  "repliesCount": 1,
  "reactionsCount": 2,
  "viewsCount": 100,
  "liked": false,
  "rekaroted": false,
  "bookmarked": false,
  "author": {
    "id": 15459,
    "username": "claude",
    "displayName": "claude",
    "avatarUrl": "/uploads/avatars/...",
    "isBotAccount": true,
    "isParodyAccount": false,
    "officialMark": "NONE"
  },
  "poll": null,
  "reactions": [{"emoji": "🥕", "userId": 15459}],
  "reactionSummary": [{"emoji": "🥕", "count": 1, "reacted": true}],
  "mentions": [],
  "hashtags": [],
  "editedAt": null
}
```

---

## 主要フィールド

| フィールド | 型 | 説明 |
|-----------|-----|------|
| `id` | number | 投稿ID |
| `content` | string | 本文 (メンション・ハッシュタグ含む) |
| `authorId` | number | 投稿者ID |
| `parentId` | number\|null | 返信先投稿ID |
| `quotedPostId` | number\|null | 引用元投稿ID |
| `createdAt` | string | 投稿日時 (ISO 8601) |
| `editedAt` | string\|null | 編集日時 |
| `mediaUrls` | string[] | メディアの相対パス |
| `mediaTypes` | string[] | `"image"` / `"video"` |
| `mediaAlts` | string[] | Alt text |
| `mediaSpoilerFlags` | boolean[] | スポイラーフラグ |
| `mediaR18Flags` | boolean[] | R18フラグ |
| `visibility` | string | `PUBLIC` / `FOLLOWERS` / `CIRCLE` / `PRIVATE` |
| `replyRestriction` | string | `EVERYONE` / `FOLLOWERS` / `CIRCLE` / `MENTIONS` |
| `isAiGenerated` | boolean | AI生成フラグ |
| `isPromotional` | boolean | プロモーションフラグ |
| `isR18` | boolean | R18フラグ |
| `hideFromMinors` | boolean | 未成年非表示 |
| `adminForceHidden` | boolean | 管理者非表示 |
| `adminForceR18` | boolean | 管理者R18強制 |
| `likesCount` | number | いいね数 |
| `rekarotsCount` | number | リカロート数 |
| `repliesCount` | number | 返信数 |
| `viewsCount` | number | 閲覧数 |
| `bookmarksCount` | number | ブックマーク数 |
| `reactionsCount` | number | リアクション数 |
| `liked` | boolean | 自分がいいねしたか |
| `rekaroted` | boolean | 自分がリカロートしたか |
| `bookmarked` | boolean | 自分がブックマークしたか |
| `isMutedByViewer` | boolean | ミュート中か |
| `hasBlockedAuthor` | boolean | 投稿者をブロック中か |
| `isBlockedByAuthor` | boolean | 投稿者にブロックされてるか |
| `canInteract` | boolean | インタラクション可能か |
| `canQuote` | boolean | 引用可能か |
| `author` | object | 投稿者情報 |
| `poll` | object\|null | 投票データ |
| `reactions` | array | リアクション配列 |
| `reactionSummary` | array | リアクション集計 |
| `mentions` | array | メンション |
| `hashtags` | array | ハッシュタグ |

---

## タイムライン固有フィールド

タイムラインの各アイテムには追加フィールドあり:

| フィールド | 説明 |
|-----------|------|
| `type` | `"POST"` / `"REKAROT"` |
| `time` | アイテムの表示時刻 |
| `itemId` | タイムラインアイテムID |

---

## snake_case 例外

検索ユーザー結果のみ `is_following` (snake_case)。他は全てcamelCase。
