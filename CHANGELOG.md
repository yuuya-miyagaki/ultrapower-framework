# Changelog — Ultrapower

## v4.2.0 (2026-03-24)

### デュアルDB 抽象化パターン

- **AGENTS.md**: Provider Pattern テンプレート追加（db.js → Adapter）
- **統一API契約**: auth(getUser/signUp/signIn/signOut) + CRUD(from/select/insert/update/delete)
- **切替方法**: `VITE_DB_PROVIDER=supabase|firebase` 環境変数
- **StudyFlow 実証**: 全ページ接続完了、83テスト PASS

### lint 修正

- **README.md**: テーブルスタイル統一 + ベアURL修正
- **AGENTS.md**: リスト前空行追加

## v4.1.1 (2026-03-24)

### レトロ改善（StudyFlow PJ 振り返りから）

1. **テスト比率向上**: 39 → 71テスト (82%増)
   - pages/ テスト追加 (dashboard, bookmarks, notes, settings)
   - キーバリデーションテスト追加 (supabase-config)

2. **キーバリデーション標準化**: `supabase.js` にJWTチェック追加
   - 3パート構造チェック
   - issuer `supabase` チェック
   - バッククォート・不正文字検出
   - UIトースト通知

3. **DB自動化手順**: AGENTS.md 更新
   - Supabase CLI マイグレーション手順追加
   - Firebase MCP セットアップ自動化手順追加

### ドキュメント整備

- **README.md**: 全面改訂（クイックスタート、典型フロー、実践結果）
- **GUIDE.md**: 新規作成（全15スキルの使い方ガイド + Tips + 安全ガードレール）

## v4.1.0 (2026-03-24)

### テスト検証で発見・適用した改善（8件）

#### Phase A — メインワークフロー

1. **ultrapower-workflow**: 新規PJルート追加（onboardスキップ→brainstorm直行）
2. **ultra-implement**: Vanilla JS UMDモジュールパターン手順追加
3. **ultra-plan**: Lightレビューモード（小規模変更向け簡易判定）
4. **ultra-qa**: Dev Server起動手順追記（静的HTML対応）
5. **AGENTS.md**: DB Noneパス追加（DB不使用PJ対応）
6. **AGENTS.md**: アーティファクト保存パスにdesign-reviews追加

#### Phase B — 独立スキルテスト

7. **ultra-brainstorm**: 新規PJで.gitignore初期生成（Step 4.5）
8. **ultra-docs**: README.mdテンプレート自動生成機能

### lint修正

- **AGENTS.md**: MD032（リスト空行）3件 + MD040（コードブロック言語指定）1件

### テスト結果サマリー

| スキル | 結果 |
|-------|------|
| ultra-brainstorm | ✅ DONE |
| ultra-plan | ✅ DONE |
| ultra-implement | ✅ DONE |
| ultra-review | ✅ DONE |
| ultra-qa | ✅ DONE |
| ultra-benchmark | ✅ A+ (TTFB 47ms, FCP 352ms) |
| ultra-design-review | ✅ 60/70 (85.7%) |
| ultra-retro | ✅ DONE_WITH_CONCERNS |
| ultra-docs | ✅ DONE_WITH_CONCERNS |
| ultra-onboard | ✅ DONE (Draw.io + Memory) |
| ultra-debug | ✅ DONE (TDD RED→GREEN) |
| ultra-ship | ✅ DONE (npm audit 0脆弱性) |
| ultra-design-system | ✅ DONE (DESIGN.md + Preview) |
| ultra-second-opinion | ✅ DONE (プロンプト生成) |

## v4.0.0 (2026-03-23)

- 初期リリース: gstack × Superpowers 統合フレームワーク
- 15スキル + 1オーケストレーター
- MCP統合: Context7, Playwright, Memory, Draw.io, GitHub, Firebase
- 完了ステータスプロトコル、安全ガードレール、See Something Say Something
