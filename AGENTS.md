# Pholio – Agent Instructions

## プロジェクト概要
カメラマン向けCRM「Pholio」。Firebase Hosting + Firestore + Vanilla JS。
本番URL: https://photocrm-app.firebaseapp.com
リポジトリ: ~/Desktop/sasukechanparis.github.io

## 技術スタック
- Frontend: Vanilla JS (app.js ~11,800行), HTML, CSS
- Backend: Firebase Firestore
- Auth: Firebase Auth (Google OAuth, signInWithPopup)

## ⚠️ SDK注意事項（最重要）
- firebase-config.js → compat SDK → `db.collection().doc().get()` 形式
- firebase-service.js → v9モジュラー → `getDoc(doc(db,...))` 形式
- 両者を絶対に混在させないこと。ファイルごとに既存パターンを確認してから修正すること。

## 作業ルール（必ず守ること）
1. 修正前に必ずファイルを読む
2. node --check [file] でシンタックス検証
3. 修正箇所以外は変更しない
4. デプロイは git push + firebase deploy --only hosting

## デプロイコマンド
```bash
cd ~/Desktop/sasukechanparis.github.io
git add -A && git commit -m "[type]: [内容]" && git push
firebase deploy --only hosting
```

## コミットメッセージ規則
feat / fix / style / refactor / docs / deploy
