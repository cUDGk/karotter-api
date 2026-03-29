# Python Bot 実装例

以下の例では認証済みの `sess` (requests.Session) と `headers` (認証ヘッダー) を使用する前提。

---

## テキスト投稿

```python
def post_karoot(text):
    r = sess.post(f"{BASE}/posts",
        data={"content": text, "visibility": "PUBLIC", "replyRestriction": "EVERYONE"},
        headers=headers)
    return r.json()["post"]["id"]

post_id = post_karoot("Hello Karotter!")
```

---

## 画像付き投稿

```python
def post_with_image(text, image_path):
    with open(image_path, "rb") as f:
        files = [("media", (image_path.split("/")[-1], f, "image/png"))]
        data = {
            "content": text,
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
            "mediaAlts": json.dumps([""]),
            "mediaSpoilerFlags": json.dumps([False]),
            "mediaR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/posts", files=files, data=data, headers=headers)
    return r.json()["post"]["id"]
```

---

## 動画付き投稿

```python
def post_with_video(text, video_path):
    with open(video_path, "rb") as f:
        files = [("media", ("video.mp4", f, "video/mp4"))]
        data = {
            "content": text,
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
            "mediaAlts": json.dumps([""]),
            "mediaSpoilerFlags": json.dumps([False]),
            "mediaR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/posts", files=files, data=data, headers=headers)
    return r.json()["post"]["id"]

# 画像と動画は同時に投稿できない（400エラー）
```

---

## リプライ

```python
def reply(text, parent_id):
    r = sess.post(f"{BASE}/posts",
        data={
            "content": text,
            "parentId": str(parent_id),
            "visibility": "PUBLIC",
            "replyRestriction": "EVERYONE",
        },
        headers=headers)
    return r.json()["post"]["id"]

reply("返信テスト", 12345)
```

---

## 投稿削除

```python
def delete_post(post_id):
    r = sess.delete(f"{BASE}/posts/{post_id}", headers=headers)
    return r.status_code == 200
```

---

## タイムライン取得

```python
def get_timeline(page=1, limit=15):
    r = sess.get(f"{BASE}/posts/timeline",
        params={"page": page, "limit": limit, "mode": "latest"},
        headers=headers)
    return r.json()["posts"]

for post in get_timeline():
    print(f"@{post['author']['username']}: {post['content'][:50]}")
```

---

## 通知取得 & 返信

```python
def get_notifications(limit=30):
    r = sess.get(f"{BASE}/notifications",
        params={"limit": limit},
        headers=headers)
    return r.json()["notifications"]

for notif in get_notifications():
    if notif["type"] == "REPLY":
        post = notif["post"]
        print(f"@{notif['actor']['username']} replied: {post['content']}")
        # reply(f"返信ありがとう", post["id"])
```

---

## いいね / リアクション

```python
def like(post_id):
    sess.post(f"{BASE}/posts/{post_id}/like", headers=headers)

def unlike(post_id):
    sess.delete(f"{BASE}/posts/{post_id}/like", headers=headers)

def react(post_id, emoji):
    sess.post(f"{BASE}/posts/{post_id}/react",
        json={"emoji": emoji}, headers=headers)

def unreact(post_id, emoji):
    from urllib.parse import quote
    sess.delete(f"{BASE}/posts/{post_id}/react/{quote(emoji)}", headers=headers)

like(12345)
react(12345, "🥕")
```

---

## フォロー / フォロバ

```python
def follow(user_id):
    sess.post(f"{BASE}/follow/{user_id}", headers=headers)

def get_followers(user_id, limit=100):
    r = sess.get(f"{BASE}/users/{user_id}/followers",
        params={"limit": limit}, headers=headers)
    return r.json()

def get_following(user_id, limit=1000):
    r = sess.get(f"{BASE}/users/{user_id}/following",
        params={"limit": limit}, headers=headers)
    return r.json()

# 自動フォロバ
def auto_followback(my_id):
    followers = {u["id"] for u in get_followers(my_id)}
    following = {u["id"] for u in get_following(my_id)}
    for uid in followers - following:
        follow(uid)
        print(f"Followed back: {uid}")
```

---

## DM送信

```python
def send_dm(group_id, text):
    r = sess.post(f"{BASE}/dm/groups/{group_id}/messages",
        data={"content": text},
        headers=headers)
    return r.json()

def send_dm_with_image(group_id, text, image_path):
    with open(image_path, "rb") as f:
        files = [("attachments", (image_path.split("/")[-1], f, "image/png"))]
        data = {
            "content": text,
            "attachmentAlts": json.dumps([""]),
            "attachmentSpoilerFlags": json.dumps([False]),
            "attachmentR18Flags": json.dumps([False]),
        }
        r = sess.post(f"{BASE}/dm/groups/{group_id}/messages",
            files=files, data=data, headers=headers)
    return r.json()

# 投稿のフィールド名は "media"、DMは "attachments" — 間違えると500エラー
```

---

## DM取得

```python
def get_dm_groups():
    r = sess.get(f"{BASE}/dm/groups", headers=headers)
    return r.json()["groups"]

def get_dm_messages(group_id, limit=50):
    r = sess.get(f"{BASE}/dm/groups/{group_id}/messages",
        params={"limit": limit}, headers=headers)
    return r.json()["messages"]
```

---

## 投票

```python
def vote(post_id, option_id):
    sess.post(f"{BASE}/posts/{post_id}/poll/vote",
        json={"optionId": option_id}, headers=headers)

# 投票データは GET /posts/{id} の post.poll に含まれる
# 再度POSTで投票取り消し（トグル式）
```

---

## Developer API (APIキー認証)

```python
API_KEY = "kar_live_xxxxx"
DEV_BASE = "https://api.karotter.com/api/developer"
dev_headers = {
    "Authorization": f"Bearer {API_KEY}",
    "X-Client-Type": "web",
    "X-Device-Id": "550e8400-e29b-41d4-a716-446655440000"
}

# タイムライン取得
r = requests.get(f"{DEV_BASE}/timeline", headers=dev_headers)
posts = r.json()["posts"]

# 投稿
r = requests.post(f"{DEV_BASE}/posts",
    json={"content": "Developer APIから投稿"},
    headers=dev_headers)

# いいね
requests.post(f"{DEV_BASE}/posts/12345/like", headers=dev_headers)

# Developer APIでは DELETE /posts/{id} (投稿削除) は使用不可
```
