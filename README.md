# kinko-app

スタッフ用金庫引継ぎ記録アプリ。

## データ管理 (Firebase)

このアプリはFirebase Authentication(メール/パスワード)とCloud Firestoreでデータを管理しています。

- `index.html` 内の `firebaseConfig`(空欄)に、Firebaseコンソール「プロジェクトの設定」→「マイアプリ」で取得できる値を貼り付けてください。
- ログインには、Firebase Authenticationの「Users」タブで発行したメールアドレス・パスワードを使用します。
- Firestore Security Rules(`firestore.rules`)は `main` ブランチへのpush時にGitHub Actionsで自動デプロイされます。デプロイには、リポジトリシークレット `FIREBASE_SERVICE_ACCOUNT`(サービスアカウントJSON)の登録、サービスアカウントへの `Service Usage Consumer` / `Firebase Rules Admin` ロールの付与、`.firebaserc` の `default` プロジェクトIDの設定が必要です。

## Firestoreデータ構造

- `meta/storeMaster` — 店舗一覧 (`stores: [{ id, name, target }]`)
- `storesConfig/{storeId}` — 店舗ごとのスタッフ一覧 (`staffList: string[]`)
- `storesConfig/{storeId}/records/{recordId}` — 店舗ごとの金庫・レジ引継ぎ記録(追記型)

## 店舗の切り替え・追加

「マスタ設定」画面から、端末の所属店舗を切り替えたり、新しい店舗を追加できます。店舗ごとに金庫設定金額・スタッフ一覧・引継ぎ記録が独立して管理されます。

## 旧GAS/スプレッドシートからのデータ移行(初回のみ)

Firebase移行前に旧システム(Google Apps Script + スプレッドシート)で運用していた店舗・スタッフ・引継ぎ記録がある場合は、`tools/migrate-from-gas.html` を使って一括移行できます。

1. `tools/migrate-from-gas.html` をブラウザで開く(GitHub Pagesで公開している場合はそのURLの`/tools/migrate-from-gas.html`)
2. kinko-appと同じFirebase Authenticationアカウントでログイン
3. 「① 旧データを取得してプレビュー」で店舗数・スタッフ数・記録数を確認
4. 内容に問題なければ「② Firestoreへ書き込みを実行」

書き込みは同じ記録に対して何度実行しても上書きになるだけで重複しません。移行が完了したらこのファイルは削除して構いません。
