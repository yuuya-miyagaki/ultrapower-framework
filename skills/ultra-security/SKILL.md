---
name: ultra-security
description: "セキュリティ監査スキル。OWASP Top 10 + STRIDE 脅威モデリング + シークレット考古学 + 依存関係監査 + MCP supply chain チェック。「セキュリティ」「脆弱性チェック」「OWASP」「セキュリティ監査」で起動。"
---

# Ultra Security — セキュリティ監査

> gstack `/cso` + antigravity-kit `vulnerability-scanner` の統合。Ultrapower のMCPエコシステムに特化。

## 起動条件

- 「セキュリティ監査」「脆弱性チェック」「OWASP」
- ultra-ship 前の最終チェック（自動提案）
- ultra-implement でセキュリティ関連コードを変更した時

> **学習パターン参照**: 同ディレクトリの `learned-patterns.md` が存在する場合、スキル実行時に参照し、過去の知見を活用する。

## 引数

- `/security` — フル監査（全フェーズ、デフォルト）
- `/security quick` — クイックスキャン（Phase 1-3 のみ、5分以内）
- `/security deps` — 依存関係のみ
- `/security secrets` — シークレット考古学のみ
- `/security owasp` — OWASP Top 10 のみ
- `/security mcp` — MCP supply chain のみ

---

## Phase 1: シークレット考古学

> **最初にインフラのドアを確認**。コードレビューよりシークレット漏洩の方がインパクトが大きい。

### 1.1 Git 履歴スキャン

```bash
# .env ファイルの履歴を確認（現在だけでなく過去のコミットも）
run_command: git log --all --diff-filter=A -- "*.env" "*.env.*" ".env*" --format="%H %s" | head -20

# シークレットパターン検出（全履歴）
run_command: git log -p --all -S "PRIVATE_KEY\|SECRET\|PASSWORD\|API_KEY\|TOKEN" -- "*.js" "*.ts" "*.json" "*.yaml" "*.yml" "*.md" | grep -E "^\+.*?(PRIVATE_KEY|SECRET|PASSWORD|API_KEY|TOKEN)" | head -30
```

### 1.2 現在のシークレット露出チェック

```bash
# ハードコードされたシークレット検出
grep_search: パターン="(sk-|ghp_|gho_|glpat-|xoxb-|xoxs-|AKIA|AIza)" 対象: "src/"
grep_search: パターン="password\s*[:=]\s*['\"][^'\"]+['\"]" 対象: "src/"

# .gitignore に .env が含まれているか
grep_search: パターン=".env" 対象: ".gitignore"
```

### 1.3 結果分類

| 深刻度 | パターン | 対応 |
|--------|---------|------|
| 🔴 P1 | 有効なAPIキー・トークンがgit履歴に存在 | 即座にローテーション + BFG Repo-Cleaner |
| 🟡 P2 | .env が .gitignore に未登録 | .gitignore に追加 **AUTO-FIX** |
| 🟢 P3 | テスト用ダミーキーがハードコード | 環境変数化を推奨 |

---

## Phase 2: 依存関係 Supply Chain 監査

### 2.1 既知の脆弱性スキャン

```bash
# npm プロジェクト
run_command: npm audit --json 2>/dev/null || echo "npm audit not available"

# Python プロジェクト
run_command: pip audit --format json 2>/dev/null || echo "pip audit not available"

# 古い依存関係の検出
run_command: npm outdated --json 2>/dev/null || echo "npm outdated not available"
```

### 2.2 Supply Chain リスク評価

```bash
# package.json の依存関係を分析
view_file: package.json

# postinstall スクリプトの確認（マルウェアの入口）
run_command: cat package.json | grep -A2 '"postinstall\|preinstall\|install"' || echo "No install hooks"

# lockfile の整合性チェック
run_command: test -f package-lock.json && echo "lockfile exists" || echo "⚠️ No lockfile - supply chain risk"
```

### 2.3 結果分類

| 深刻度 | 条件 | 対応 |
|--------|------|------|
| 🔴 P1 | Critical/High 脆弱性 | 即座にアップデート |
| 🟡 P2 | Moderate 脆弱性 | 計画的にアップデート |
| 🟢 P3 | Low 脆弱性 / outdated | 次回スプリントで対応 |

---

## Phase 3: MCP Supply Chain 監査

> **Ultrapower 独自フェーズ**: MCP エコシステム特有のリスクを評価。

### 3.1 MCP 設定チェック

```bash
# MCP サーバー一覧の確認
run_command: cat ~/.gemini/antigravity/mcp_config.json 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); print(json.dumps(list(d.get('mcpServers',{}).keys()), indent=2))" 2>/dev/null || echo "MCP config not found"
```

### 3.2 MCP リスク評価チェックリスト

| チェック項目 | 検証方法 |
|-------------|---------|
| 各MCPサーバーの認証情報が安全に管理されているか | 環境変数 vs ハードコード |
| MCP サーバーのバージョンが最新か | npm list で確認 |
| 不要なMCPサーバーが有効になっていないか | `mcp_budget.md` と照合 |
| MCPサーバーが過剰な権限を持っていないか | allowed-tools / 実行可能コマンドを確認 |
| サードパーティMCPの信頼性 | npm パッケージのダウンロード数・メンテナンス状況 |

### 3.3 Skill Supply Chain（ultra-* スキル）

```bash
# スキルファイルの改ざん検出（直近の変更）
run_command: git log --since="7 days" --format="%H %s" -- "skills/*/SKILL.md" | head -10

# スキルが参照する外部リソースのチェック
grep_search: パターン="(https?://|npm install|pip install)" 対象: "skills/"
```

---

## Phase 4: OWASP Top 10 コードレビュー

> **Context7 MCP で最新の OWASP ガイドラインを自動参照**してからレビュー開始。

### 4.1 Context7 最新ガイドライン取得

```yaml
mcp_context7_resolve-library-id:
  libraryName: "OWASP"
  query: "OWASP Top 10 web application security risks"

# 結果に応じて最新ドキュメントを参照
mcp_context7_query-docs:
  libraryId: "[取得したID]"
  query: "Top 10 security vulnerabilities prevention"
```

### 4.2 チェックリスト

| # | OWASP カテゴリ | チェック対象 | 検証コマンド |
|---|---------------|-------------|-------------|
| A01 | アクセス制御の不備 | 認証チェックの有無、認可ロジック | `grep_search` で認証ミドルウェア確認 |
| A02 | 暗号化の失敗 | HTTP vs HTTPS、パスワードハッシュ | `grep_search` で `http://` / 平文パスワード |
| A03 | インジェクション | SQL組立、XSS、コマンドインジェクション | `grep_search` で `innerHTML` / テンプレートリテラル内ユーザー入力 |
| A04 | 安全でない設計 | レート制限、入力検証の欠如 | アーキテクチャレビュー |
| A05 | セキュリティ設定ミス | デバッグモード本番稼働、CORS設定 | `grep_search` で `debug: true` / `cors: *` |
| A06 | 脆弱・古いコンポーネント | Phase 2 で検出済み | npm audit 結果を参照 |
| A07 | 認証の失敗 | セッション管理、ブルートフォース対策 | 認証フロー分析 |
| A08 | ソフトウェアとデータの整合性 | CI/CDの安全性、依存関係の検証 | Phase 2 & Phase 3 参照 |
| A09 | ログと監視の不備 | エラーログ、監査ログ | `grep_search` で `console.log` / ロギングライブラリ |
| A10 | SSRF | 外部URL取得、リダイレクト | `grep_search` で `fetch` / `axios` + 動的URL |

### 4.3 コード自動スキャン

```bash
# XSS: innerHTML 使用箇所
grep_search: パターン="innerHTML\s*=" 対象: "src/"

# SQLインジェクション: テンプレートリテラル内のクエリ
grep_search: パターン="(SELECT|INSERT|UPDATE|DELETE).*\$\{" 対象: "src/"

# CORS: ワイルドカード設定
grep_search: パターン="(cors|Access-Control).*\*" 対象: "src/"

# eval/Function: コードインジェクション
grep_search: パターン="(eval\(|new Function\()" 対象: "src/"

# Firestore/Supabase セキュリティルール
run_command: cat firestore.rules 2>/dev/null || cat supabase/migrations/*.sql 2>/dev/null | grep -i "GRANT\|POLICY" | head -20
```

---

## Phase 5: STRIDE 脅威モデリング（フル監査時）

> `/security quick` ではスキップ。

| 脅威 | 分析対象 | 質問 |
|------|---------|------|
| **S**poofing（なりすまし） | 認証フロー | 他ユーザーになりすませるか？ |
| **T**ampering（改ざん） | データフロー | 入力・保存データを改ざんされるか？ |
| **R**epudiation（否認） | ログ | 行動の否認が可能か？（監査ログは？） |
| **I**nformation Disclosure | データ保存 | 意図しないデータ露出はあるか？ |
| **D**enial of Service | API | レート制限なしでリソース枯渇するか？ |
| **E**levation of Privilege | 認可 | 一般ユーザーが管理者操作できるか？ |

---

## Phase 6: LLM / AI セキュリティ（該当する場合）

> プロジェクトが LLM/AI を使用している場合のみ実行。

| チェック項目 | リスク | 対応 |
|-------------|-------|------|
| プロンプトインジェクション | ユーザー入力がシステムプロンプトに混入 | 入力サニタイズ + 出力検証 |
| LLM 出力の無検証利用 | 生成コードの直接実行、SQLの直接実行 | サンドボックス + バリデーション |
| API キーの露出 | フロントエンドにAPIキーがハードコード | サーバーサイドプロキシ |
| トークン消費攻撃 | 大量入力でコスト爆発 | 入力長制限 + レート制限 |

---

## 完了レポート

### レポート保存

```bash
run_command: mkdir -p docs/security-reports
```

`docs/security-reports/YYYY-MM-DD-security-audit.md` に保存。

### セキュリティポスチャーレポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-SECURITY 完了                     ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  監査モード:    [full/quick/deps/...]    ║
║  Phase 1 シークレット:  [N] 件検出       ║
║  Phase 2 依存脆弱性:    [N] 件検出       ║
║  Phase 3 MCP リスク:    [N] 件検出       ║
║  Phase 4 OWASP:         [N] 件検出       ║
║  Phase 5 STRIDE:        [N] 脅威特定     ║
║  Phase 6 LLM:           [N] 件検出       ║
╠══════════════════════════════════════════╣
║  🔴 P1 (即座に修正):    [N] 件           ║
║  🟡 P2 (計画的に修正):  [N] 件           ║
║  🟢 P3 (改善推奨):      [N] 件           ║
║  次のフェーズ:  ultra-ship               ║
╚══════════════════════════════════════════╝
```

### 深刻度ルール

- **P1 が 1件以上** → STATUS: `DONE_WITH_CONCERNS`、ultra-ship を BLOCKED
- **P2 のみ** → STATUS: `DONE_WITH_CONCERNS`、ultra-ship は条件付き許可
- **P3 のみ** → STATUS: `DONE`、ultra-ship に進行可能

### ultra-ship との連携

ultra-ship 実行時に `docs/security-reports/` の最新レポートを自動参照。
P1 未解決の場合は出荷を阻止する。

---

## レッドフラグ

- ❌ セキュリティ問題を「低リスク」と過小評価する
- ❌ シークレットの具体的な値をレポートに書く（ファイル名と行番号のみ）
- ❌ 「問題なし」と報告して検証コマンドの出力を示さない
- ❌ Phase 3（MCP）をスキップする（Ultrapower の独自リスク領域）
