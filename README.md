# ニッポンゲッサー

**公開URL：https://kota422531.github.io/nippon-guesser/**

Googleストリートビューで「ここは日本のどこ？」を当てる、2人対戦のWebアプリです。

- 出題は日本国内だけ（全47都道府県＋沖縄・奄美の離島）
- ホストが部屋を作り、招待リンク（LINE／コピー／共有）を相手に送れば対戦できる
- ラウンド数（3・5・10）、制限時間（60・120・180秒・なし）、移動あり／なしを選べる
- 得点は1ラウンド最大5,000点。日本の広さに合わせて、10km差で約4,700点、150km差で約1,840点
- 構成：GitHub Pages（静的ホスティング）＋ Firebase Firestore（対戦状態の同期）＋ Google Maps JavaScript API

---

## セットアップ手順

### 1. Google Maps APIキーを作る
1. [Google Cloud Console](https://console.cloud.google.com/) でプロジェクトを作成
2. 「お支払い」で請求先アカウントを設定（ストリートビューを使うには必須。月ごとの無料枠があります）
3. 「APIとサービス → ライブラリ」で **Maps JavaScript API** を有効化
4. 「認証情報 → 認証情報を作成 → APIキー」でキーを作成
5. 作ったキーに制限をかける（**必ずやってください**。キーはページ内に見える状態になるため）
   - アプリケーションの制限：**ウェブサイト**
     - `https://kota422531.github.io/*`
     - 手元で試すなら `http://localhost:*/*` も追加
   - APIの制限：**Maps JavaScript API** のみ
6. 使いすぎ防止のため、「APIとサービス → 割り当て」で1日の上限を設定するか、「お支払い → 予算とアラート」で予算アラートを設定

> 料金・無料枠の最新情報は Google Maps Platform の料金ページで確認してください。
> 1ゲームあたり、出題地点探し（ストリートビューのメタデータ検索）と、2人分のストリートビュー表示・地図表示が発生します。

### 2. Firebaseを準備する
1. [Firebaseコンソール](https://console.firebase.google.com/) で新しいプロジェクトを作成（例：`nippon-guesser`）
2. 「Firestore Database」を作成（ロケーションは `asia-northeast1`（東京）がおすすめ）
3. 「ルール」タブに `firestore.rules` の中身を貼り付けて公開
   - 既存プロジェクト（家計簿など）を使い回す場合は、既存ルールを消さずに `match /geoRooms/{roomId}` のブロックだけ追加してください
4. 「プロジェクトの設定 → マイアプリ → ウェブアプリを追加」で設定値（`firebaseConfig`）をコピー

### 3. index.html に設定を書き込む
`index.html` の先頭近くにある `CONFIG` を書き換えます。

```js
const CONFIG = {
  GOOGLE_MAPS_API_KEY: "（手順1のAPIキー）",
  firebase: {
    apiKey: "...",
    authDomain: "...",
    projectId: "...",
    storageBucket: "...",
    messagingSenderId: "...",
    appId: "...",
  },
};
```

### 4. GitHub Pagesで公開する
1. GitHubに `nippon-guesser` リポジトリ（Public）を作成
2. `index.html`・`firestore.rules`・`README.md` をアップロード
3. 「Settings → Pages」で `main` ブランチ / `(root)` を選んで保存
4. 数分後に `https://kota422531.github.io/nippon-guesser/` で遊べます

---

## 遊び方
1. 名前を入れて、ラウンド数などを選び「部屋を作って招待する」
2. 表示された招待リンクをLINEなどで相手に送る
3. 相手が参加したら「ゲーム開始」
4. 景色を見て「地図で回答」→ 地図をタップしてピン → 「この場所で決定」
5. 2人とも回答すると結果が表示される。どちらかが「次のラウンドへ」を押すと進む
6. 最後に合計点で勝敗。「もう一度対戦する」で新しい出題で再戦

## 補足
- 出題地点は「各都道府県の主要地点の周辺からランダムに選び、半径15km以内で一番近い公式ストリートビュー」を拾う方式です。山奥や海の上は出ません
- 同じゲーム内で出題地点どうしは80km以上離れるようにしています
- 対戦データは Firestore の `geoRooms` コレクションに残ります。気になる場合は Firestore の TTL ポリシー（`createdAt` 基準）で自動削除できます
- 時間制限「なし」で相手がページを閉じると、そのラウンドは進めなくなります（もう一度リンクを開けば続きから再開できます）
