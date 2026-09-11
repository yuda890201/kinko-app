# kinko-app

スタッフ用金庫引継ぎ記録アプリ。

## データ管理 (Firebase)

このアプリはFirebase Authentication(店舗共通PINコード方式)とCloud Firestoreでデータを管理しています。

- `index.html` 内の `firebaseConfig`(空欄)に、Firebaseコンソール「プロジェクトの設定」→「マイアプリ」で取得できる値を貼り付けてください。
- Firestore Security Rules(`firestore.rules`)は `main` ブランチへのpush時にGitHub Actionsで自動デプロイされます。デプロイには、リポジトリシークレット `FIREBASE_SERVICE_ACCOUNT`(サービスアカウントJSON)の登録、サービスアカウントへの `Service Usage Consumer` / `Firebase Rules Admin` ロールの付与、`.firebaserc` の `default` プロジェクトIDの設定が必要です。

### ログイン(店舗共通PINコード)

スタッフはメールアドレスの入力なしで、PINコード(数字)だけでログインできます。裏側では、固定のダミーメールアドレス宛の「店舗共通アカウント」をFirebase Authenticationに1つ登録し、そのパスワードをPINコードとして扱っています。

セットアップ手順:
1. Firebase Console → Authentication → Sign-in method で「メール/パスワード」を有効化
2. Authentication → Users → 「ユーザーを追加」で、以下の内容で1件だけ登録する
   - メールアドレス: `staff@kinko-app-2f5e4.local`(`index.html` 内の `SHARED_LOGIN_EMAIL` と同じ値)
   - パスワード: 運用したいPINコード(Firebaseの制約で **6文字以上** が必要)
3. PINコードを変更したい場合は、Firebase ConsoleのUsers画面からこのアカウントのパスワードを再設定するだけでよい(コードの変更は不要)

以前作成していた個人ごとのメール/パスワードアカウントは、このアプリからは使われなくなったため削除して構いません。

## Firestoreデータ構造

- `meta/storeMaster` — 店舗一覧 (`stores: [{ id, name, target }]`)
- `storesConfig/{storeId}` — 店舗ごとのスタッフ一覧 (`staffList: string[]`)
- `storesConfig/{storeId}/records/{recordId}` — 店舗ごとの金庫・レジ引継ぎ記録(追記型)

## 店舗の切り替え・追加

「マスタ設定」画面から、端末の所属店舗を切り替えたり、新しい店舗を追加できます。店舗ごとに金庫設定金額・スタッフ一覧・引継ぎ記録が独立して管理されます。

## 旧GAS/スプレッドシートからのデータ移行

旧システム(Google Apps Script + スプレッドシート)からのデータ移行は完了済みです(移行用ツールは使用後に削除済み)。
