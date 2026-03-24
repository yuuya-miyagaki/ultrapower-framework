---
name: ultra-ship
description: "出荷準備完了時に起動。セキュリティ監査 + 開発完了フロー（4オプション）+ PR作成 + デプロイ。「出荷」「PR作って」「デプロイ」で起動。"
---

# Ultra Ship — セキュリティ監査 + 完了フロー + PR + デプロイ

> gstack ship + cso + Superpowers finishing-a-development-branch の統合

## 起動条件

- 「出荷して」「PRを作って」「デプロイして」
- ultra-qa が全チェック通過した後

## Step 1: Pre-flight チェック

### 1.1 ブランチ確認

```bash
run_command: git rev-parse --abbrev-ref HEAD
```text

main/masterブランチなら **中止**: 「featureブランチから出荷してください」

### 1.2 デフォルトブランチ検出

```bash
# デフォルトブランチ名を動的検出
run_command: git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main"
```text

以下の `<default>` は上記で検出したブランチ名。

### 1.3 変更の把握

```bash
run_command: git status
run_command: git diff <default>...HEAD --stat
run_command: git log <default>..HEAD --oneline
```bash

### 1.4 ベースブランチをマージ

```bash
run_command: git fetch origin <default> && git merge origin/<default> --no-edit
```text

コンフリクトが発生 → 自動解決を試行 → 複雑なら **停止**

### 1.5 テスト実行（マージ後のコードで）

```bash
run_command: [テスト実行コマンド]
```text

テスト失敗 → **停止** → ultra-debug を推奨

## Step 2: セキュリティ監査（CSO統合）

### 依存関係監査

```bash
# 依存パッケージの脆弱性チェック（各言語を独立実行）
run_command: npm audit 2>/dev/null; true
run_command: pip-audit 2>/dev/null; true
run_command: cargo audit 2>/dev/null; true
```text

> **注:** 各コマンドは独立実行。未インストールのツールはスキップされる。混合言語プロジェクトでも全言語を漏れなく監査。

| 重大度 | 対応 |
| ------ | ---- |
| Critical/High | **停止** — 修正必須 |
| Medium | 警告 + ユーザーに報告 |
| Low | レポートのみ |

### OWASP Top 10 チェック

| # | 脆弱性 | チェック方法 |
|---|--------|-------------|
| A01 | アクセス制御の不備 | 認可チェック漏れの検出。`grep_search` で `isAdmin` / `role` / `auth` ガード欠如を確認 |
| A02 | 暗号化の失敗 | `grep_search: "password\|secret\|api_key\|token"` でハードコードされたシークレット検索 |
| A03 | インジェクション | `grep_search: "exec(\|eval(\|child_process\|\\$\\{"` でコマンド/テンプレートインジェクション検出。SQL文字列結合は `grep_search: "\\+ .*query\|\\` + .*sql\|f\".*SELECT"` で検出 |
| A04 | 安全でない設計 | 脅威モデリング |
| A05 | セキュリティ設定ミス | デフォルト設定の確認 |
| A06 | 脆弱なコンポーネント | 依存関係の監査 |
| A07 | 認証の失敗 | セッション管理の確認 |
| A08 | データ整合性の失敗 | デシリアライゼーション攻撃 |
| A09 | ログ/監視の不足 | セキュリティイベントのログ確認 |
| A10 | SSRF | サーバー側リクエストの検証 |

### STRIDE 脅威モデリング

```text
THREAT          | ASSET           | MITIGATION      | STATUS
----------------|-----------------|------------------|--------
Spoofing        | [認証]          | [対策]           | [✓/✗]
Tampering       | [データ]        | [対策]           | [✓/✗]
Repudiation     | [監査]          | [対策]           | [✓/✗]
Info Disclosure | [機密データ]    | [対策]           | [✓/✗]
DoS             | [可用性]        | [対策]           | [✓/✗]
Elevation       | [権限]          | [対策]           | [✓/✗]
```text

### 偽陽性除外リスト

以下は自動的にFPとして除外:
1. テストファイル内のハードコード値
2. 開発環境専用の設定
3. ローカルホスト参照
4. 例示用のダミーデータ
5. 型安全な ORM クエリ
6. フレームワーク内蔵のCSRF保護
7. 環境変数からのシークレット読み込み
8. CI/CD パイプラインの設定値

### 信頼度ゲート

セキュリティ監査の信頼度が **8/10 未満** → ユーザーに報告して確認を求める

## Step 3: 開発完了フロー（Superpowers finishing-a-branch 統合）

### 3.1 テスト最終確認

```bash
run_command: [テスト実行コマンド]
```text

テスト失敗 → Step 3禁止。修正必須。

### 3.2 完了4オプション

```text
実装完了。どうしますか？

1. <default>にローカルマージ
2. プッシュしてPRを作成（推奨）
3. ブランチをそのまま保持
4. この作業を破棄
```bash

### Option 1: ローカルマージ

```bash
git checkout <default>
git pull
git merge [feature-branch]
[テスト実行]
git branch -d [feature-branch]
```bash

### Option 2: PR作成（推奨）

```bash
git push -u origin [feature-branch]
```bash

PR本文を自動生成:
- 変更概要（git log から）
- セキュリティ監査結果
- テスト結果
- QAレポート（利用可能な場合）

```bash
# GitHub MCP で PR作成
mcp_github_create_pull_request
```text

### Option 3: ブランチ保持

報告のみ。クリーンアップなし。

### Option 4: 破棄

**必ず確認**: 「この操作は不可逆です。'discard' と入力してください。」

```bash
git checkout <default>
git branch -D [feature-branch]
```bash

### Worktree クリーンアップ（Option 1, 2, 4）

```bash
# worktree内で作業している場合
git worktree list | grep [feature-branch]
git worktree remove [worktree-path]
```text

## Step 4: デプロイ（オプション）

ユーザーが要求した場合のみ:

### DB バックエンド対応デプロイ

**Supabase プロジェクト:**
```bash
# マイグレーション適用
run_command: supabase db push

# フロントエンドデプロイ（Vercel等）
run_command: vercel deploy --prod
```yaml

**Firebase プロジェクト:**

> **⚠️ 安全ガードレール（AGENTS.md 参照）**
> `firebase deploy` は本番環境に影響するコマンドです。実行前にユーザーの明示的な確認を取得してください。

```bash
# Firebase MCP でデプロイ
mcp_firebase-mcp-server_firebase_init: hosting + firestore
run_command: firebase deploy
```text

### Web アプリ確認

```text
mcp_playwright_browser_navigate → デプロイ先URL
mcp_playwright_browser_snapshot → ページ構造確認
mcp_playwright_browser_console_messages → エラー確認
mcp_playwright_browser_take_screenshot → デプロイ確認
```text

### ポストデプロイ検証

- [ ] ページが正常に読み込まれる
- [ ] コンソールエラーなし
- [ ] 主要機能が動作する
- [ ] APIが応答する
- [ ] DB接続が正常（Supabase/Firebase）

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-SHIP 完了                         ║
║  STATUS: [DONE / BLOCKED]                ║
╠══════════════════════════════════════════╣
║  セキュリティ監査: [PASS/ISSUES]         ║
║  依存監査:     [Critical:N High:N]       ║
║  OWASP Top 10:    [N]/10 チェック済み    ║
║  STRIDE:          [N]/6 対策済み         ║
║  テスト:          [PASS/FAIL]            ║
║  DB:              [Supabase/Firebase/N/A] ║
║  完了オプション:  [選択されたオプション]   ║
║  PR URL:          [URL or N/A]           ║
║  次のフェーズ: ultra-retro              ║
╚══════════════════════════════════════════╝
```
