# kinko-app

スタッフ用金庫引継ぎ記録アプリ。

## データ管理 (Firebase)

このアプリはFirebase Authentication(店舗ごとの専用PINコード方式)とCloud Firestoreでデータを管理しています。

- `index.html` 内の `firebaseConfig`(空欄)に、Firebaseコンソール「プロジェクトの設定」→「マイアプリ」で取得できる値を貼り付けてください。
- Firestore Security Rules(`firestore.rules`)は `main` ブランチへのpush時にGitHub Actionsで自動デプロイされます。デプロイには、リポジトリシークレット `FIREBASE_SERVICE_ACCOUNT`(サービスアカウントJSON)の登録、サービスアカウントへの `Service Usage Consumer` / `Firebase Rules Admin` ロールの付与、`.firebaserc` の `default` プロジェクトIDの設定が必要です。

### ログイン(店舗ごとの専用PINコード + オーナーPIN)

スタッフはメールアドレスの入力なしで、自店舗のPINコードだけでログインできます。ログインすると自動的にその店舗に固定され、店舗切替・新規追加・金庫設定金額の変更メニューは表示されません(他店舗のデータを誤って触れないようにするため)。

全店舗を横断できる「オーナーPIN」も別に存在し、オーナーPINでログインした場合のみ、従来通り店舗の切替・新規追加・設定金額の変更ができます。

**仕組み**: 店舗ごと・オーナー用に、固定のダミーメールアドレス宛のFirebase Authenticationアカウントを1つずつ登録し、そのパスワードをPINコードとして扱っています。ログイン画面はPINだけを受け取り、`meta/storeLoginDirectory`(店舗ID→ログイン用メールアドレスの対応表、公開読み取り専用)から候補メールアドレス一覧を取得したうえで、「オーナー→各店舗」の順に入力されたPINでサインインを試行し、最初に成功したアカウントに応じて店舗(またはオーナー権限)を自動確定します。

**新しい店舗を追加する手順**:
1. オーナーPINでログインし、「マスタ設定」画面から新規店舗を追加する(`meta/storeMaster` にstoreIdが発行される)
2. Firebase Console → Authentication → Users → 「ユーザーを追加」で、その店舗専用のダミーメールアドレス(例: `store-xxxxxx@<authDomain>`)とPIN(**6文字以上**必須)を登録する
3. Firebase Console → Firestore Database → データ タブで `meta/storeLoginDirectory` ドキュメントを開き、`stores` マップに `{ 発行されたstoreId: 2で登録したメールアドレス }` を追加する

**PINを変更したい場合**は、Firebase ConsoleのAuthentication → Usersから該当アカウントのパスワードを再設定するだけでよい(コード・Firestore側の変更は不要)。

**オーナーアカウントのセットアップ**(初回のみ): Authentication → Users で、`index.html` 内の `OWNER_LOGIN_EMAIL` と同じメールアドレスを1件登録し、パスワードをオーナーPINとする。

過去に使われていた店舗共通アカウント(`staff@<authDomain>`)や個人ごとのメール/パスワードアカウントは、このアプリからは使われなくなったため削除して構いません。

## Firestoreデータ構造

- `meta/storeMaster` — 店舗一覧 (`stores: [{ id, name, target }]`)
- `meta/storeLoginDirectory` — 店舗ID→ログイン用メールアドレスの対応表 (`stores: { [storeId]: loginEmail }`)。ログイン前でも読み取れるよう公開読み取り。書き込みはオーナーのみ
- `storesConfig/{storeId}` — 店舗ごとのスタッフ一覧 (`staffList: string[]`)
- `storesConfig/{storeId}/records/{recordId}` — 店舗ごとの金庫・レジ引継ぎ記録(追記のみ。更新・削除不可)

## 権限モデル(Firestore Security Rules)

- 読み取り: ログイン済みであれば誰でも可(閲覧専用アカウントを含む)
- 書き込み: オーナーは全店舗、店舗専用アカウントは自店舗の `storesConfig/{storeId}` 以下のみ
- `meta/{doc}`(`storeMaster`など)の書き込みはオーナーのみ
- `records` サブコレクションは `create` のみ許可(`update`/`delete`は常に拒否)し、過去の引継ぎ記録を改ざんできないようにしている

外部アプリ(店舗コミュニケーションアプリなど)向けに、書き込み権限を持たない閲覧専用アカウントを別途発行できます(`isSignedIn()`を満たせば読み取り可能なため)。

## 店舗の切り替え・追加

オーナーPINでログインした場合のみ、「マスタ設定」画面から端末の所属店舗を切り替えたり、新しい店舗を追加できます。店舗ごとに金庫設定金額・スタッフ一覧・引継ぎ記録が独立して管理されます。店舗専用PINでログインした場合は、常にその店舗に固定されます。

## 旧GAS/スプレッドシートからのデータ移行

旧システム(Google Apps Script + スプレッドシート)からのデータ移行は完了済みです(移行用ツールは使用後に削除済み)。
