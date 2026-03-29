# 管理パネル (Admin) エンドポイント

管理者向けの全エンドポイント (`/control-room-x9k2/*`)。

---

## 概要

| 項目 | 詳細 |
|------|------|
| フロントエンドURL | `https://karotter.com/admin` |
| APIベースパス | `/control-room-x9k2` |
| JSチャンク | `Admin-BX1OyPEg.js` |
| 認証 | `user.isAdmin === true` が必要 |
| 非管理者アクセス | 「アクセス拒否 -- 管理者権限が必要です」表示 |
| 全APIレスポンス | `403: 管理者権限が必要です` |
| 確認済みエンドポイント | 136 |

---

## UIタブ構成 (JSチャンクから確認)

| タブID | ラベル | 説明 |
|--------|--------|------|
| `dashboard` | ダッシュボード | システム統計 (総ユーザー数、総カロート数、24h活動ユーザー、未対応通報) |
| `users` | ユーザー管理 | 検索、BAN/解除、フラグ変更、公式マーク付与、アカウント編集、削除 |
| `posts` | カロート管理 | 検索、R18強制、非表示、削除 |
| `stories` | ストーリー管理 | 検索、R18強制、非表示、削除、有効期限表示 |
| `recommend` | おすすめ β | おすすめアルゴリズムテスト (userId指定、スコア詳細表示) |
| `trending` | トレンド β | トレンドアルゴリズムテスト (velocity, uniqueActors, engagementRate) |
| `survey` | アンケート | おすすめ満足度アンケート結果 (投票数、満足度スコア) |
| `beta-experiment` | Beta実験 | A/Bテスト結果 (A〜E 5バリアント、userId%5で振り分け) |

---

## ダッシュボード統計

```
GET /control-room-x9k2/analytics

Response:
{
  "totalUsers": 12000,
  "totalPosts": 120000,
  "activeUsers": 2300,     // 24時間アクティブ
  "totalReports": 5        // 未対応通報数
}
```

---

## ユーザー管理

管理者がユーザーに対して行える操作:

### アカウント情報変更

`PATCH /control-room-x9k2/users/{id}/account`:
- username、displayName の変更
- email 変更（任意）
- password 強制リセット（任意）
- emailVerified フラグの切り替え

### 公式マーク

`PATCH /control-room-x9k2/users/{id}/official-mark`:

| 値 | ラベル |
|-----|--------|
| `YELLOW` | 黄色: 認証済み団体マーク |
| `PURPLE` | 紫: 運営マーク |
| `BLUE` | 青: 本人マーク |
| `GRAY` | 灰色: 政府関係者マーク |
| `BLACK` | 黒: 認証済みマーク |
| `RED` | 赤: 認証済みマーク |
| `GREEN` | 緑: 認証済みマーク |
| `ORANGE` | オレンジ: 認証済みマーク |

- 複数マーク同時付与可能（配列で管理）

### フラグ操作

`PATCH /control-room-x9k2/users/{id}/flags`:

| フラグ | 説明 |
|--------|------|
| `isParodyAccount` | パロディアカウント |
| `isBotAccount` | BOTユーザー |
| `adminForceHidden` | 管理者非表示 |
| `adminForceParody` | パロディ強制 |
| `adminForceBot` | BOT強制 |
| `showBotAccounts` | BOT表示許可 |
| `showHiddenPosts` | 非表示閲覧許可 |
| `showR18Content` | R18表示許可 |
| `hideProfileFromMinors` | 未成年にプロフ非表示 |

### BAN

`PATCH /control-room-x9k2/users/{id}/ban`:
- BAN時に `reason` を指定
- BAN中ユーザーには赤い「BAN中」バッジ表示
- `unban` で解除

### 削除

`DELETE /control-room-x9k2/users/{id}`:
- 確認ダイアログ:「@{username} を削除しますか？この操作は取り消せません。」

---

## 投稿管理

管理者が投稿に対して行える操作:
- `adminForceR18`: R18強制フラグ
- `adminForceHidden`: 非表示フラグ
- 投稿削除

表示情報: viewsCount、likesCount、rekarotsCount、repliesCount

---

## ストーリー管理

投稿管理と同様のフラグ操作に加え:
- mediaType（画像/動画）のプレビュー表示
- caption表示
- visibility（PUBLIC/FOLLOWERS/サークル）
- 有効期限（expiresAt）の表示

---

## おすすめアルゴリズムテスト (β)

```
GET /control-room-x9k2/test-recommend?limit=30&userId={id}

Response:
{
  "targetUser": { "id", "username", "displayName" },
  "followingCount": 100,
  "candidateCount": 500,
  "signalsSummary": {
    "authorAffinityCount": 50,
    "socialProofCount": 30,
    "mutualFollowCount": 20,
    "tagAffinityTop10": [{"tag": "プログラミング", "score": 85}]
  },
  "rawTopByScore": [{"rank", "author", "score", "trending"}],
  "ranked": [{
    "rank", "author", "content",
    "stats": {"likes", "rekarots", "replies", "views"},
    "ageHours",
    "scores": {
      "engagement", "freshness", "authorAffinity",
      "graph", "socialProof", "tagAffinity",
      "inNetwork", "totalRecommend"
    }
  }]
}
```

### A/Bテスト 5バリアント

| タイプ | コンセプト | 新しさ | エンゲージメント | 会話 | 発見 |
|--------|-----------|--------|----------------|------|------|
| A | 新しさ最優先 | ★★★ | ★ | - | - |
| B | 質重視 | ★ | ★★★ | ★ | - |
| C | バランス | ★★ | ★★ | ★ | ★ |
| D | 会話重視 | ★★ | ★★ | ★★★ | - |
| E | 発見重視 | ★★ | ★★ | ★ | ★★★ |

ユーザーは `userId % 5` でバリアントに自動振り分け。

---

## トレンドアルゴリズムテスト (β)

```
GET /control-room-x9k2/test-trending?limit=30

Response:
{
  "candidateCount": 200,
  "rawTop10": [{"rank", "author", "total", "velocity", "uniqueActors"}],
  "ranked": [{
    "rank", "author", "content",
    "stats": {"likes", "rekarots", "replies", "views"},
    "ageHours",
    "trendBreakdown": {
      "velocity", "uniqueActors", "engagementRate", "freshness", "total"
    },
    "windowStats": {
      "likes1h", "likes6h", "likes24h",
      "rekarots1h", "rekarots6h", "rekarots24h",
      "replies1h", "replies6h", "replies24h",
      "uniqueActors"
    }
  }]
}
```

---

## 全エンドポイント一覧

### コア (3)

```
GET /control-room-x9k2
GET /control-room-x9k2/dashboard
GET /control-room-x9k2/overview
```

### ユーザー管理 (22)

```
GET    /control-room-x9k2/users
GET    /control-room-x9k2/users/{id}
GET    /control-room-x9k2/users/search
POST   /control-room-x9k2/users
PATCH  /control-room-x9k2/users/{id}/ban
PATCH  /control-room-x9k2/users/{id}/unban
PATCH  /control-room-x9k2/users/{id}/verify
PATCH  /control-room-x9k2/users/{id}/flags
PATCH  /control-room-x9k2/users/{id}/account
PATCH  /control-room-x9k2/users/{id}/official-mark
PATCH  /control-room-x9k2/users/{id}/email
PATCH  /control-room-x9k2/users/{id}/password
PATCH  /control-room-x9k2/users/{id}/role
GET    /control-room-x9k2/users/{id}/sessions
GET    /control-room-x9k2/users/{id}/posts
GET    /control-room-x9k2/users/{id}/reports
GET    /control-room-x9k2/users/{id}/bans
GET    /control-room-x9k2/users/{id}/notes
GET    /control-room-x9k2/users/{id}/history
GET    /control-room-x9k2/users/{id}/suspend
GET    /control-room-x9k2/users/{id}/restrict
POST   /control-room-x9k2/users/{id}/warn
DELETE /control-room-x9k2/users/{id}/delete
```

### 投稿管理 (6)

```
GET    /control-room-x9k2/posts
GET    /control-room-x9k2/posts/{id}
GET    /control-room-x9k2/posts/search
PATCH  /control-room-x9k2/posts/{id}/flags
PATCH  /control-room-x9k2/posts/{id}/hide
DELETE /control-room-x9k2/posts/{id}/delete
```

### ストーリー管理 (2)

```
GET    /control-room-x9k2/stories
PATCH  /control-room-x9k2/stories/{id}/flags
```

### 通報 / モデレーション (11)

```
GET  /control-room-x9k2/reports
GET  /control-room-x9k2/reports/pending
GET  /control-room-x9k2/reports/resolved
GET  /control-room-x9k2/reports/{id}
POST /control-room-x9k2/reports/{id}/resolve
POST /control-room-x9k2/reports/{id}/dismiss
POST /control-room-x9k2/reports/{id}/escalate
GET  /control-room-x9k2/moderation
GET  /control-room-x9k2/moderation/queue
GET  /control-room-x9k2/moderation/filters
GET  /control-room-x9k2/moderation/automod
GET  /control-room-x9k2/moderation/words
GET  /control-room-x9k2/moderation/rules
GET  /control-room-x9k2/flagged-content
```

### BAN (6)

```
GET  /control-room-x9k2/bans
GET  /control-room-x9k2/bans/{id}
POST /control-room-x9k2/bans/create
GET  /control-room-x9k2/ip-bans
GET  /control-room-x9k2/ip-bans/{id}
GET  /control-room-x9k2/shadowbans
```

### コンテンツ管理 (15)

```
GET  /control-room-x9k2/badges
GET  /control-room-x9k2/badges/{id}
POST /control-room-x9k2/badges/create
GET  /control-room-x9k2/emoji
GET  /control-room-x9k2/emoji/{id}
POST /control-room-x9k2/emoji/create
GET  /control-room-x9k2/frames
GET  /control-room-x9k2/frames/{id}
GET  /control-room-x9k2/gacha
GET  /control-room-x9k2/gacha/{id}
GET  /control-room-x9k2/gacha/items
GET  /control-room-x9k2/themes
GET  /control-room-x9k2/themes/{id}
GET  /control-room-x9k2/stickers
GET  /control-room-x9k2/stickers/{id}
```

### お知らせ (3)

```
GET  /control-room-x9k2/announcements
GET  /control-room-x9k2/announcements/{id}
POST /control-room-x9k2/announcements/create
```

### アナリティクス (11)

```
GET /control-room-x9k2/analytics
GET /control-room-x9k2/analytics/users
GET /control-room-x9k2/analytics/posts
GET /control-room-x9k2/analytics/growth
GET /control-room-x9k2/analytics/engagement
GET /control-room-x9k2/analytics/retention
GET /control-room-x9k2/analytics/revenue
GET /control-room-x9k2/stats
GET /control-room-x9k2/stats/users
GET /control-room-x9k2/stats/posts
GET /control-room-x9k2/stats/daily
```

### 申請 / リクエスト (3)

```
GET /control-room-x9k2/verification-requests
GET /control-room-x9k2/bot-requests
GET /control-room-x9k2/appeals
```

### システム (16)

```
GET  /control-room-x9k2/settings
POST /control-room-x9k2/settings
GET  /control-room-x9k2/config
GET  /control-room-x9k2/system
GET  /control-room-x9k2/logs
GET  /control-room-x9k2/logs/{type}
GET  /control-room-x9k2/audit
GET  /control-room-x9k2/audit-log
GET  /control-room-x9k2/maintenance
POST /control-room-x9k2/maintenance
GET  /control-room-x9k2/backup
POST /control-room-x9k2/backup
GET  /control-room-x9k2/cron
GET  /control-room-x9k2/queue
GET  /control-room-x9k2/tasks
GET  /control-room-x9k2/jobs
GET  /control-room-x9k2/cache
POST /control-room-x9k2/cache/clear
GET  /control-room-x9k2/database
GET  /control-room-x9k2/migrations
```

### その他管理 (38)

```
GET /control-room-x9k2/beta-experiment
GET /control-room-x9k2/test-recommend
GET /control-room-x9k2/test-trending
GET /control-room-x9k2/survey-results
GET /control-room-x9k2/feedback
GET /control-room-x9k2/media
GET /control-room-x9k2/uploads
GET /control-room-x9k2/notifications
GET /control-room-x9k2/invites
GET /control-room-x9k2/features
GET /control-room-x9k2/feature-flags
GET /control-room-x9k2/roles
GET /control-room-x9k2/permissions
GET /control-room-x9k2/radio
GET /control-room-x9k2/draw
GET /control-room-x9k2/dm
GET /control-room-x9k2/emails
GET /control-room-x9k2/actions
GET /control-room-x9k2/sessions
GET /control-room-x9k2/webhooks
GET /control-room-x9k2/api-keys
GET /control-room-x9k2/apikeys
GET /control-room-x9k2/premium
GET /control-room-x9k2/subscriptions
GET /control-room-x9k2/monetization
GET /control-room-x9k2/payments
GET /control-room-x9k2/domains
GET /control-room-x9k2/rate-limits
GET /control-room-x9k2/blocked-words
GET /control-room-x9k2/filtered-words
GET /control-room-x9k2/trending
GET /control-room-x9k2/trending/override
GET /control-room-x9k2/search
GET /control-room-x9k2/search/index
```

---

## 管理パネルUIプレビュー (Chrome拡張)

非管理者でも管理パネルのUIだけを閲覧できるChrome拡張機能。
APIは依然403を返すため、データは取得できない（UI構造の確認のみ）。

### 仕組み

Karotter本体の `Admin-*.js` チャンクへのリクエストを、改変版JSにリダイレクトする。
改変箇所は `isAdmin` チェックの無効化（`enabled: !!(null==_?void 0:_.isAdmin)` → `enabled: true`）のみ。

### ファイル構成

**manifest.json**
```json
{
  "manifest_version": 3,
  "name": "Karotter Admin",
  "version": "1.1",
  "permissions": ["declarativeNetRequest"],
  "host_permissions": ["https://karotter.com/*"],
  "declarative_net_request": {
    "rule_resources": [{
      "id": "ruleset_1",
      "enabled": true,
      "path": "rules.json"
    }]
  },
  "web_accessible_resources": [
    {
      "resources": ["admin.js"],
      "matches": ["https://karotter.com/*"]
    }
  ]
}
```

**rules.json**
```json
[
  {
    "id": 1,
    "priority": 1,
    "action": {
      "type": "redirect",
      "redirect": {
        "extensionPath": "/admin.js"
      }
    },
    "condition": {
      "urlFilter": "https://karotter.com/assets/Admin-*.js",
      "resourceTypes": ["script", "xmlhttprequest"]
    }
  }
]
```

**admin.js**
- 本物の `Admin-*.js` チャンクのコピー
- `enabled: !!(null==_?void 0:_.isAdmin)` を `enabled: true` に変更（全てのReact Queryクエリの `enabled` 条件）
- これにより非管理者でもUIコンポーネントがレンダリングされる

### インストール方法

1. 3ファイルを同じフォルダに配置
2. `chrome://extensions` を開く
3. 「デベロッパーモード」ON
4. 「パッケージ化されていない拡張機能を読み込む」でフォルダを選択
5. `https://karotter.com/admin` にアクセス

### 注意事項

- **APIデータは取得不可** — 全エンドポイントが403を返すため、UIは表示されるがデータは空
- **チャンクハッシュが変わると動作しなくなる** — Karotterのデプロイ毎にJSファイル名が変わるため、`rules.json` の `urlFilter` と `admin.js` のimportパスを更新する必要がある
- **常時有効だとフロント挙動に影響** — 確認後は拡張機能を無効化推奨
