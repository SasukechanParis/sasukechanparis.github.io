# Pholio Agent Team セットアップガイド
**作成日:** 2026年3月8日

---

## 全体像

```
あなた（CEO・亮祐）
    ↓ タスクを渡す
Codex CLI（GPT-5.4）
    ↓ AGENTS.mdを読んで実行
    ├── planner   → 計画を立てる
    ├── implementer → コードを書く
    ├── tester    → 動作確認手順を出す
    └── reviewer  → コードをレビューする
```

各roleは**手動で切り替える**。自動連携ではなく、あなたが指示する。

---

## Step 1: Codex CLI インストール

```bash
# インストール
npm install -g @openai/codex

# バージョン確認
codex --version

# APIキー設定（OpenAIダッシュボードで発行: https://platform.openai.com/api-keys）
export OPENAI_API_KEY="sk-..."
echo 'export OPENAI_API_KEY="sk-..."' >> ~/.zshrc
source ~/.zshrc
```

---

## Step 2: プロジェクトにファイルを配置

```bash
cd ~/Desktop/sasukechanparis.github.io

# AGENTS.md と agents/ フォルダを作成
mkdir -p agents

# 以下のファイルをダウンロードフォルダからコピー
cp ~/Downloads/AGENTS.md ./AGENTS.md
cp ~/Downloads/HANDOFF_2026_03_08.md ./HANDOFF_2026_03_08.md
```

agentsフォルダに以下4ファイルを作成（内容は下記参照）：
```
agents/
  planner.md
  implementer.md
  tester.md
  reviewer.md
```

```bash
# git に追加
git add -A && git commit -m "docs: add AGENTS.md and agent team files" && git push
```

---

## Step 3: agents/ フォルダの中身

### agents/planner.md
```markdown
# Planner Agent

## 役割
機能要件を受け取り、実装計画に分解する。コードは書かない。

## 出力フォーマット
### タスク分解
1. [ファイル名] 修正内容（行番号）
2. [ファイル名] 修正内容（行番号）

### 影響範囲
- 変更するファイル
- 変更しないファイル
- テストすべき動作

### 優先度
P0（即時）/ P1（今週）/ P2（将来）

## Pholio固有ルール
- firebase-config.js と firebase-service.js のSDKを確認してから計画する
- Firestoreルール変更は必ずfirebase deployを含める
- スタッフ権限に関わる変更はstaffEmailsインデックスへの影響を確認する
```

### agents/implementer.md
```markdown
# Implementer Agent

## 役割
Plannerの計画に従いコードを実装する。

## 必須手順
1. 対象ファイルを必ず読み込んでから編集開始
2. node --check [file] で構文確認
3. SDKパターン確認（firebase-config.js=compat / firebase-service.js=v9）
4. 修正箇所以外は絶対に変更しない

## デプロイコマンド
cd ~/Desktop/sasukechanparis.github.io
git add -A && git commit -m "[type]: [内容]" && git push
firebase deploy --only hosting
```

### agents/tester.md
```markdown
# Tester Agent

## 役割
実装後の動作確認手順を生成し、コンソールエラーを診断する。

## スタッフ権限テスト手順
1. オーナー(sasuke.photographe@gmail.com)でログイン
2. 設定→チーム管理を開く
3. ログアウト
4. スタッフ(koppu.purin@gmail.com)でログイン
5. コンソール確認:
   ✅ [STAFF] owner settings loaded
   ✅ [GET] isStaff: true
   ✅ [GET] loaded: N clients (N>0)
   ❌ Missing or insufficient permissions → ルール確認

## Firebase Console確認パス
users/qaNGdaqc5lXGgxvj8ZSNUeYg0Mp2/meta/staffEmails
→ emails配列にスタッフのメールがあるか

## 無視してよいエラー
- Cross-Origin-Opener-Policy（ポップアップ認証の副作用）
- LaunchDarkly initialized（機能フラグ正常動作）
```

### agents/reviewer.md
```markdown
# Reviewer Agent

## 役割
実装済みコードの品質・セキュリティ・一貫性をレビューする。

## チェックリスト
### セキュリティ
- [ ] スタッフが他オーナーのデータにアクセスできないか
- [ ] staffEmailsが正しく更新されるか
- [ ] オーナー以外がstaffEmailsを書き込めないか

### コード品質
- [ ] firebase-config.js と firebase-service.js のSDK混在がないか
- [ ] node --check でエラーがないか
- [ ] finally ブロックでUI強制表示していないか

### UX
- [ ] スタッフ画面で売上・経費が非表示か
- [ ] 屋号/スタッフ名がヘッダーに表示されるか
- [ ] リリース対象外言語がセレクターに表示されていないか

## 出力フォーマット
✅ 問題なし
⚠️ 軽微な問題（動作に影響なし）
❌ 要修正（動作に影響あり）
```

---

## Step 4: 実際の使い方

### 基本フロー
```bash
cd ~/Desktop/sasukechanparis.github.io

# ① まずHandoffを読ませる（初回だけ）
codex "HANDOFF_2026_03_08.mdとAGENTS.mdを読んでプロジェクトを把握して。概要を教えて"

# ② Plannerとして計画を立てさせる
codex "agents/planner.mdを読んでから、プラン料金の通貨換算機能の実装計画を立てて"

# ③ Implementerとして実装させる
codex "agents/implementer.mdとAGENTS.mdを読んでから、[計画内容]を実装して"

# ④ Testerとして確認手順を出させる
codex "agents/tester.mdを読んでから、今回の変更のテスト手順を教えて"

# ⑤ Reviewerとしてレビューさせる
codex "agents/reviewer.mdを読んでから git diff HEAD~1 をレビューして"
```

### コピペ用：最初に投げるプロンプト
```
HANDOFF_2026_03_08.mdとAGENTS.mdを読んでプロジェクト全体を把握してください。
その上で、P1タスクの①「プラン料金の通貨換算」を実装してください。
app.jsのCURRENCY_CONFIG（81行付近）とformatPlanMonthlyPrice（1187行付近）を修正。
修正前に必ずファイルを読み、完了後にnode --check app.jsを実行してください。
```

### コピペ用：言語リリースフラグ実装
```
HANDOFF_2026_03_08.mdを読んでください。
セクション7「言語リリースフラグ機能」を実装してください。
admin_config/releaseFlagsからリリース済み言語を取得し、
言語セレクターに未リリース言語を表示しない処理をapp.jsに追加。
EUユーザー検知バナーも追加してください。
```

### コピペ用：ストレージモード切り替え実装
```
HANDOFF_2026_03_08.mdのセクション6を読んでください。
ストレージモード切り替え機能を実装してください。
まずisCloudMode()関数とsaveClient/fetchClientsの抽象レイヤーだけ実装して、
動作確認後に同意UIを追加する方針で進めてください。
```

### コピペ用：利用規約・プライバシーポリシー作成
```
HANDOFF_2026_03_08.mdのセクション8を読んでください。
terms.html と privacy.html を作成してください。
- 日本語・英語の2言語
- 対象地域は現時点では日本
- クラウド同期の免責事項を含む
- EU圏ユーザーへの注記を含む（準備中）
- 既存のstyle.cssのデザインに合わせる
```

---

## Step 5: よくある使い方パターン

### バグ修正
```bash
codex "AGENTS.mdを読んでから以下のエラーを修正して：[コンソールログ貼り付け]"
```

### 新機能追加
```bash
# まず計画
codex "agents/planner.mdを読んで [機能名] の実装計画を立てて。コードは書かないで"
# 計画を確認してOKなら実装
codex "agents/implementer.mdとAGENTS.mdを読んで、上記計画を実装して"
```

### コードレビュー
```bash
codex "agents/reviewer.mdを読んでから git diff HEAD~1 をレビューして"
```

### デプロイ
```bash
git add -A && git commit -m "feat: [内容]" && git push && firebase deploy --only hosting
```

---

## Step 6: Codex CLI vs Claude Code の使い分け

| 作業 | Codex CLI (GPT-5.4) | Claude Code |
|------|---------------------|-------------|
| 長いファイルの編集 | ✅ 得意 | ✅ 得意 |
| Firebase Console操作 | ❌ 不可 | ❌ 不可（手動） |
| 計画・設計相談 | ✅ | ✅ |
| エラー診断 | ✅ | ✅ |
| 引き継ぎ文書作成 | ✅ | ✅ |

**基本的にどちらでやっても同じ。** 
今使っているツールで続けるのが最速。

---

*このドキュメントはClaude Sonnet 4.6が2026年3月8日に作成。*
