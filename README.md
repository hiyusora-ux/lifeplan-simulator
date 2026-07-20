# ライフプラン シミュレーター（ログイン付き・家族向け / Firebase）

ID（メールアドレス）とパスワードでログインでき、入力データが**クラウドに保存されて複数端末で同期**される構成です。
認証は **Firebase Authentication**（車両アプリと同じ方式）、データ保存は **Cloud Firestore**、ホスティングは **Vercel**。ビルド不要の静的サイトです。
※このアプリ専用に **新しい Firebase プロジェクト**を作ります（車両アプリとはデータ完全分離）。

## ファイル構成
- `index.html` … ログイン画面＋アプリの外枠（Firebase認証）
- `simulator.html` … シミュレーター本体
- `config.js` … Firebaseの接続情報（**あなたが編集**）
- `README.md` … この手順書

---

## セットアップ手順

### A. Firebase プロジェクトを新規作成
1. https://console.firebase.google.com にアクセス（Googleアカウントでログイン）
2. **プロジェクトを追加** → 名前（例：`lifeplan`）を入力して作成
   （Googleアナリティクスは任意。無しでOK）

### B. ログイン（メール/パスワード）を有効化
1. 左メニュー **構築 → Authentication** → **始める**
2. **Sign-in method** タブ → **メール/パスワード** を選び **有効にする** → 保存

### C. Firestore（データベース）を作成
1. 左メニュー **構築 → Firestore Database** → **データベースの作成**
2. ロケーションを選択（例：`asia-northeast1`）→ **本番環境モード**で開始
3. **ルール**タブを開き、内容を以下に置き換えて **公開**：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 自分のプランは自分だけが読み書きできる
    match /plans/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

### D. ウェブアプリを登録して接続情報を取得
1. 左上 **プロジェクトの概要（⚙ → プロジェクトの設定）** → 下の「マイアプリ」で **ウェブ `</>`** を選択
2. アプリのニックネーム（例：`lifeplan-web`）を入力して登録（Hostingは不要）
3. 表示される `firebaseConfig`（apiKey などの塊）を控える

### E. config.js に貼り付け
`config.js` を開き、Dで控えた値に置き換えます：

```js
window.FIREBASE_CONFIG = {
  apiKey:            "...",
  authDomain:        "lifeplan-xxxx.firebaseapp.com",
  projectId:         "lifeplan-xxxx",
  storageBucket:     "lifeplan-xxxx.appspot.com",
  messagingSenderId: "...",
  appId:             "..."
};
```

### F. 家族向けの設定（おすすめ）
- **Authentication → Settings → ユーザーアクション** で、家族の登録が済んだら
  **「アカウントの作成（サインアップ）を有効にする」をオフ**にすると、他人が勝手に登録できなくなります。
  追加したいときは **Authentication → Users → ユーザーを追加** から管理者が作成できます。
- あとで公開URLが決まったら、**Authentication → Settings → 承認済みドメイン**に
  Vercelのドメイン（`＊＊＊.vercel.app`）を **追加**してください（これが無いとログインできません）。

---

## G. GitHub + Vercel で公開

### G-1. GitHub にアップ
`index.html` `simulator.html` `config.js` `README.md` の**4ファイル**をリポジトリにアップロード → Commit。

（コマンドライン派）
```bash
cd lifeplan-web
git init && git add . && git commit -m "初版"
git branch -M main
git remote add origin https://github.com/<ユーザー名>/lifeplan-simulator.git
git push -u origin main
```

### G-2. Vercel でデプロイ
1. https://vercel.com に **Continue with GitHub** でログイン
2. **Add New… → Project** → リポジトリを **Import**
3. Framework Preset は **Other** のまま **Deploy**
4. 発行された `https://＊＊＊.vercel.app` を開くとログイン画面が出ます 🎉

### G-3. 仕上げ
手順Fのとおり、発行URLを Firebase の **承認済みドメイン**に追加してください。

---

## 使い方
1. 公開URLを開く → **新規登録**でメールアドレス（＝ログインID）とパスワードを設定
2. 以降はログインすると、前回の入力が自動で復元されます
3. 入力を変えると数百ミリ秒後にクラウドへ自動保存（右上に「✓ クラウドに保存しました」）

## セキュリティ補足
- `config.js` の apiKey などは公開前提の識別情報です。**データの安全性はFirestoreルール（手順C）で守ります**。ルールは消さないでください。
- 「ID」はメールアドレスを使います（車両アプリと同じ）。任意のユーザー名にしたい場合は追加実装が必要です。
- パスワードのハッシュ化・セッション管理・再設定メールなどはFirebaseが安全に処理します。
