# Socket.IO (リアルタイム通信)

**接続先**: `wss://karotter.com/socket.io`

---

## DM

| イベント | 方向 | 説明 |
|---------|------|------|
| `dd.join` | C→S | DMグループ参加 |
| `dd.leave` | C→S | DMグループ退出 |
| `dd.read` | C→S | 既読マーク |
| `dd.newMessage` | S→C | 新規メッセージ受信 |
| `dd.messageDeleted` | S→C | メッセージ削除 |
| `dd.messageUpdated` | S→C | メッセージ更新 |
| `dd.memberAdded` | S→C | メンバー追加 |
| `dd.memberLeft` | S→C | メンバー退出 |
| `dd.memberRemoved` | S→C | メンバー削除 |
| `dd.requestUpdated` | S→C | DMリクエスト更新 |

---

## 通話 (Voice)

| イベント | 方向 | 説明 |
|---------|------|------|
| `voice:offer` | C→S | WebRTC SDP Offer |
| `voice:answer` | C→S | WebRTC SDP Answer |
| `voice:ice-candidate` | C→S | ICE candidate |
| `voice:hangup` | C→S | 通話切断 |
| `voice:participant-state` | C→S | マイク/スピーカー状態 |
| `fd.offer` | S→C | SDP Offer受信 |
| `fd.answer` | S→C | SDP Answer受信 |
| `fd.iceCandidate` | S→C | ICE candidate受信 |
| `fd.hangup` | S→C | 切断通知 |
| `fd.participantState` | S→C | 参加者状態変更 |
| `pd.state` | S→C | 通話状態変更 |
| `pd.incoming` | S→C | 着信通知 |

---

## スクリーンシェア

| イベント | 方向 | 説明 |
|---------|------|------|
| `screen-share:view` | C→S | 画面共有視聴 |

---

## ラジオ/スペース

| イベント | 方向 | 説明 |
|---------|------|------|
| `radio:signal` | C→S | WebRTCシグナリング |
| `radio:renegotiate-request` | C→S | 再ネゴシエーション |
| `radio:participant-state` | C→S | 参加者状態 |
| `vd.join` | S→C | スペース参加通知 |
| `vd.leave` | S→C | スペース退出通知 |
| `vd.userJoined` | S→C | ユーザー参加 |
| `vd.userLeft` | S→C | ユーザー退出 |
| `vd.ended` | S→C | スペース終了 |
| `vd.signal` | S→C | WebRTCシグナル受信 |

---

## その他

| イベント | 方向 | 説明 |
|---------|------|------|
| `notification` | S→C | プッシュ通知 |
| `hd.status` | S→C | ユーザーオンライン状態変更 |
| `draw:join` | C→S | 絵チャルーム参加 |
| `draw:leave` | C→S | 絵チャルーム退出 |
| `draw:stroke` | C↔S | 描画ストローク |
| `draw:cursor` | C↔S | カーソル位置 |
| `draw:layer-sync` | C↔S | レイヤー同期 |
| `draw:chat` | C↔S | 絵チャ内チャット |
| `draw:room-state` | S→C | 絵チャルーム状態 |
| `draw:user-left` | S→C | ユーザー退出通知 |
| `draw:error` | S→C | エラー通知 |
| `radio:message` | C↔S | スペース内メッセージ |
| `radio:reaction` | C↔S | スペース内リアクション |
