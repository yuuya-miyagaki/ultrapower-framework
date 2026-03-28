# Changelog — Ultrapower

> バージョニング基準: [SemVer](https://semver.org/lang/ja/) 準拠
>
> - **Major** (X.0.0): 破壊的変更（スキルインターフェース変更、AGENTS.md大規模改定）
> - **Minor** (x.Y.0): 機能追加（新スキル、新ステップ、新機能）
> - **Patch** (x.y.Z): バグ修正、ドキュメント修正、lint修正

## [自動進化] v4.5.0 — 2026-03-28

### 構造的一貫性 + ドキュメント品質の最終仕上げ

フレームワーク全体（16スキル + ドキュメント3ファイル）の網羅的レビューから抽出した改善を適用。

#### 構造的一貫性修正（C）

- `find_by_name`（非存在ツール）参照を全スキルから排除 → `mcp_filesystem_search_files` / `grep_search` に移行
- 旧「Claude Code二次意見」表記を「別AI二次意見ブリッジ」に統一
- Playwright MCP を README 要件でオプション表記に変更
- 全16スキルに `learned-patterns.md` ファイルを生成（ヘッダーのみ含む）

#### ドキュメント品質向上（D）

- GUIDE.md に ultra-parallel の説明を追加
- GUIDE.md のセットアップ手順に `.agents/` シンボリックリンク設定の具体的コマンド例を追記
- README.md 実践結果に Small World マルチエージェントAI プロジェクトを追加

#### スキル強化（S）

- ultra-onboard: `find_by_name` → `mcp_filesystem_search_files` に移行（5箇所）
- ultra-retro: Global Step のハードコードパス → 環境変数 `ULTRAPOWER_PROJECTS_DIR` 優先 + フォールバック
- ultra-design-review / ultra-design-system / ultra-docs / ultra-parallel / ultra-second-opinion: `find_by_name` → `mcp_filesystem_search_files` / `grep_search` に移行

#### 教訓の体系化（E）

- AGENTS.md に「テスト-実装の契約同期」教訓を追加（learned-patterns から昇格）
- AGENTS.md に「バニラJS 状態管理」パターンを追加

---

## [自動進化] v4.4.0 — 2026-03-28

### ブラウザ操作デュアルモード対応（全スキル）

Playwright MCP をオプション化し、未搭載時は `browser_subagent` で自動フォールバックする体制を確立。

#### スキル更新（8ファイル）

- **ultra-qa**: E2Eテスト・レスポンシブテスト・エラーリカバリのブラウザ操作をデュアル対応
- **ultra-implement**: UI検証（クリックフロー/フォーム確認）のブラウザ操作をデュアル対応
- **ultra-benchmark**: ページパフォーマンスデータ収集のブラウザ操作をデュアル対応
- **ultra-design-review**: ファーストインプレッション・ページ巡回・修正確認のブラウザ操作をデュアル対応
- **ultra-design-system**: 競合ビジュアルリサーチ・プレビュー表示のブラウザ操作をデュアル対応
- **ultra-debug**: ブラウザデバッグ操作をデュアル対応
- **ultra-ship**: デプロイ後Web検証のブラウザ操作をデュアル対応
- **ultrapower-workflow**: ブラウザ操作セクションをツール非依存の一般化表現に変更

#### MCP予算管理

- **mcp_budget.md**: 新規作成（ツール使用数 75/100、退避サーバー管理）
- Playwright MCP / sequential-thinking を overflow に退避
- オンデマンドスワップ運用ルールを AGENTS.md に明記

## [自動進化] v4.3.1 — 2026-03-28

### MCP 不具合診断 & 文言修正

- **AGENTS.md**: Memory MCP「非搭載」→「接続不良時のフォールバック」に修正
  - 診断手順（ps確認→再起動→設定確認）を追加
  - 根本原因: プロセスは起動しているが Antigravity デーモンとの接続が切断
- **AGENTS.md**: browser_subagent セクションを「ブラウザ代替」→「ブラウザ操作」に修正
  - Playwright MCP と browser_subagent の選択基準テーブル追加
  - 両ツール共存の前提で記述を統一

## [自動進化] v4.3.0 — 2026-03-28

### 学習元: Small World（マルチエージェントAI組織プラットフォーム）

#### Antigravity 環境対応

- **AGENTS.md**: ワークスペース制約セクション追加（MCP filesystem vs run_command の使い分けガイド）
- **AGENTS.md**: Memory MCP 接続不良時のフォールバック永続化手段（JSON + learned-patterns.md + KI）
- **AGENTS.md**: `browser_subagent` ガイド追加（Playwright MCP の Antigravity 代替）

#### テスト品質強化

- **ultra-qa**: `learned-patterns.md` 新規作成
  - テスト-実装乖離の自動検出チェックリスト（定数同期/リトライロジック/配列操作/API応答形式）
  - 本番データ E2E テスト禁止ルール

#### レビュー品質強化

- **ultra-review**: `learned-patterns.md` 新規作成
  - ホットスポット駆動リファクタリング警告（変更5回超で分割提案）
  - Cross-AI レビューの有効性確認パターン

#### レトロ改善

- **ultra-retro**: Step 10 / Auto Step 5 の Memory MCP 永続化をオプショナル化
  - 環境検出ロジック追加（Memory MCP 利用可否の自動判定）
  - 利用不可の場合は JSON + learned-patterns.md で代替

## v4.2.0 (2026-03-24)

### Hugging Face Pro 統合

- **AGENTS.md**: Hugging Face（Pro）セクション拡張
  - ユースケース駆動テーブル（8パターン: Inference API / Models Hub / Datasets Hub / Spaces / JS SDK）
  - Models Hub / Datasets Hub / Spaces の探索ガイド追加
  - 「探索先行」原則との連動
- **README.md / GUIDE.md**: HF 説明を AGENTS.md と整合

### デュアルDB 抽象化パターン

- **AGENTS.md**: Provider Pattern テンプレート追加（db.js → Adapter）
- **統一API契約**: auth(getUser/signUp/signIn/signOut) + CRUD(from/select/insert/update/delete)
- **切替方法**: `VITE_DB_PROVIDER=supabase|firebase` 環境変数
- **実プロジェクト実証**: 全ページ接続完了、83テスト PASS

### 最終監査 — 構造的整合性修正

- **C1**: `ultra-retro` ベアコードブロック4箇所に言語指定追加
- **C3**: 7スキル（qa/debug/ship/implement/review/benchmark/onboard）に `learned-patterns.md` 参照行追加
- **M1**: `ultra-parallel` の説明改善（AGENTS.md）
- **M2**: `GUIDE.md` に `ultra-parallel` スキル追加
- **M3**: `README.md` スキル数を「16スキル + 1オーケストレーター」に修正
- **M4**: `README.md` スキル一覧・独立スキル・v4新機能に `ultra-parallel` 追加
- **M5**: `ultra-second-opinion` を Claude Code 固定から汎用AI対応に変更
- **M8**: デュアルDB セクションに「高度な使用法」注記追加
- **P1**: `.gitignore` 拡充（`.env*`, `node_modules`, `*.log`）
- **P2**: `studyflow.md` 本番URLに時点注記追加
- **P5**: `ultra-design-review` 保存先パスを `design-reviews/` に統一

### lint 修正

- **README.md**: テーブルスタイル統一 + ベアURL修正
- **AGENTS.md**: リスト前空行追加
- **GUIDE.md / README.md**: ベアコードブロックに `text` 言語指定追加

## v4.1.1 (2026-03-24)

### レトロ改善（実プロジェクト振り返りから）

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

1. **ultra-brainstorm**: 新規PJで.gitignore初期生成（Step 4.5）
2. **ultra-docs**: README.mdテンプレート自動生成機能

### lint修正

- **AGENTS.md**: MD032（リスト空行）3件 + MD040（コードブロック言語指定）1件

### テスト結果サマリー

| スキル | 結果 |
| ------- | ------ |
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
