---
name: ultra-qa
description: "レビュー通過後に起動。自動テスト + Playwright MCP ブラウザQA + 検証チェックリスト。"
---

# Ultra QA — 統合QAテスト

## 前提ルール（AGENTS.md「5つの鉄則」参照）

```text
検証コマンドの出力なしに「動作する」と言ってはならない
```text

「多分動く」「はず」「おそらく」はすべて虚偽。コマンドを実行し、出力を読み、結果を報告する。

## Step 1: 自動テスト実行

```bash
# フルテストスイート実行
npm test  # / pytest / cargo test / go test ./...
```yaml

確認事項:
- 全テスト通過
- 警告なし
- エラーなし

**テストが失敗した場合 → ultra-debug に遷移。QAを続行してはならない。**

## Step 2: プロジェクトタイプ判定

| タイプ | 判定基準 | QAフロー |
| ------ | -------- | -------- |
| **Webアプリ** | index.html, React, Next.js, Vue等のUI | Step 3 + Step 4 + Step 5 |
| **API/バックエンド** | REST/GraphQL, Express, FastAPI等 | Step 4 + Step 5（テストのみ） |
| **CLI/ライブラリ** | npm package, CLI tool, 純ロジック | Step 5（テストのみ） |

非Webプロジェクトの場合、Step 3（ブラウザQA）は **スキップ** する。

## Step 2.5: Dev Server 起動（Webアプリの場合）

ブラウザQAの前に、dev server を自動検出・起動する:

### 起動コマンド検出ルール

| 検出対象 | 起動コマンド |
|-----------|-----------|
| `package.json` に `scripts.dev` | `npm run dev` |
| `package.json` に `scripts.start` | `npm start` |
| `index.html` がルートに存在 | `npx -y serve . -l 3456` |
| Pythonプロジェクト | `python -m http.server 3456` |
| 上記いずれも該当しない | ユーザーに起動方法を確認 |

### 実行手順

```bash
# 1. 起動コマンド検出
grep_search: "dev\|start" 対象: package.json

# 2. バックグラウンドで起動
run_command: npm run dev  # or 検出したコマンド

# 3. 起動確認（ポート番号取得）
# 出力から localhost:XXXX を確認
```text

> **ポート番号**: デフォルトの 3000 / 5173 / 8080 等を確認。競合時は別ポートを使用。

## Step 3: ブラウザQA（Webアプリの場合）

Playwright MCP でユーザーフローを検証：

### ユーザーフロー検証

```yaml
1. mcp_playwright_browser_navigate → http://localhost:3000
2. mcp_playwright_browser_snapshot → インタラクティブ要素を確認
3. mcp_playwright_browser_fill_form → テストデータ入力
   例: [{name: "Email", type: "textbox", ref: "@e3", value: "test@example.com"}]
4. mcp_playwright_browser_click → 送信ボタン（refを指定）
5. mcp_playwright_browser_snapshot → 変化を確認（ダッシュボード表示等）
6. mcp_playwright_browser_take_screenshot → 結果をスクリーンショット
```text

### レスポンシブテスト（AGENTS.md 定義に準拠）

```text
1. mcp_playwright_browser_resize(375, 812) → モバイル
2. mcp_playwright_browser_take_screenshot → mobile.png
3. mcp_playwright_browser_resize(768, 1024) → タブレット
4. mcp_playwright_browser_take_screenshot → tablet.png
5. mcp_playwright_browser_resize(1920, 1080) → デスクトップ
6. mcp_playwright_browser_take_screenshot → desktop.png
```text

### エラーチェック

```yaml
1. mcp_playwright_browser_console_messages(level: "error") → コンソールエラー
2. mcp_playwright_browser_network_requests(includeStatic: false) → ネットワーク失敗
```text

### フォームバリデーションテスト

```text
1. mcp_playwright_browser_click → 空でsubmitボタンをクリック
2. mcp_playwright_browser_snapshot → エラーメッセージ表示を確認
3. mcp_playwright_browser_fill_form → 正しいデータ入力
4. mcp_playwright_browser_click → 再送信
5. mcp_playwright_browser_snapshot → エラー消去・成功状態を確認
```text

## Step 4: パフォーマンスベンチマーク（オプション）

簡易的なページロード時間を確認。**詳細な分析が必要な場合は ultra-benchmark スキルを使用**（Core Web Vitals、バンドルサイズ、回帰検出等に対応）。

```text
mcp_playwright_browser_navigate → 対象ページ
mcp_playwright_browser_evaluate → performance.getEntriesByType('navigation')[0]?.loadEventEnd
```text

## Step 5: 検証チェックリスト

### 完了前チェック

- [ ] 全自動テストが通過（コマンド出力で確認）
- [ ] 主要ユーザーフローがブラウザで動作（スクリーンショット証拠）
- [ ] コンソールエラーなし
- [ ] ネットワークエラーなし
- [ ] レスポンシブレイアウト正常（Webの場合）
- [ ] フォームバリデーション動作（Webの場合）
- [ ] エッジケース（空入力、長文、特殊文字）対応

### 証拠ルール（ultra-review Step 1 と共通基準）

| 主張 | 必要な証拠 | 不十分な証拠 |
| ---- | ---------- | ------------ |
| テスト通過 | テストコマンド出力: 0 failures | 前回の実行結果 |
| UI正常 | スクリーンショット | 「見た目は大丈夫」 |
| エラーなし | console_messages の出力 | 「エラーは出てない」 |
| レスポンシブ | 3サイズのスクリーンショット | 「対応済み」 |

## Step 6: バグ発見時の処理

**QA→debug→QA ループは最大3回まで。**

QA中にバグを発見した場合：

1. **バグレポート作成** — 再現手順、スクリーンショット、コンソール出力
2. **失敗するテスト作成** — バグを再現する自動テスト
3. **修正** — TDDサイクル（RED → GREEN → REFACTOR）
4. **再検証** — 修正後にQAを再実行
5. **回帰テスト自動生成** — 今後同じバグが発生しないことを保証

### ループ制限

- **1回目** — 通常のバグ修正→QA再実行
- **2回目** — 根本原因の再調査を実施してからQA再実行
- **3回目** — 修正→QA再実行の最終試行
- **4回目以降** → **ユーザーにエスカレーション**。アーキテクチャ自体の問題を議論

## 完了レポート

### レポート保存

```bash
run_command: mkdir -p docs/ultrapower/qa-reports
```text

QAレポートを `docs/ultrapower/qa-reports/YYYY-MM-DD-qa-report.md` に保存。
ultra-ship が PR本文に含めるため、ファイルとして永続化が必要。

```text
╔══════════════════════════════════════════╗
║  ULTRA-QA 完了                           ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  テスト結果:    [PASS/FAIL]              ║
║  実行テスト:    [N] 件                   ║
║  検出バグ:      [N] 件                   ║
║  修正済み:      [N] 件                   ║
║  QA-Debugループ: [N] 回                  ║
║  次のフェーズ:  ultra-ship               ║
╚══════════════════════════════════════════╝
```
