---
name: ultra-parallel
description: "並列実行・サブエージェント委譲スキル。複数の独立タスクを同時実行する。「並列で」「同時に」「サブエージェントに」で起動。パターン: parallel-investigate/parallel-build/browser-delegate/bg-pipeline。"
---

# Ultra Parallel — 並列実行 + サブエージェント委譲

> Superpowers dispatching-parallel-agents + subagent-driven-development の Antigravity 統合版

## 起動条件

- 2つ以上の独立タスクが存在する時
- 「並列でやって」「同時に進めて」「ブラウザで確認しながら」
- ultra-plan / ultra-implement の実行中に並列化すべき場面を検出した時

## Antigravity 並列メカニズム

本スキルは Antigravity 環境の **3つの並列実行メカニズム** を活用する。

### メカニズム一覧

```yaml
1. 並列ツールコール:
   説明: 同一ターンで複数ツールを同時実行
   用途: ファイル並列読込、複数検索の同時実行
   制約: 同一ファイルへの並列書き込み禁止
   ツール: view_file, grep_search, mcp_filesystem_search_files 等

2. browser_subagent:
   説明: ブラウザ操作専用サブエージェント
   用途: UI検証、スクリーンショット、E2Eテスト
   制約: 一度に1つのサブエージェントのみ
   ツール: browser_subagent

3. バックグラウンドコマンド:
   説明: run_command で非同期実行、command_status で監視
   用途: ビルド、テスト、デプロイの非同期実行
   制約: WaitMsBeforeAsync で非同期化が必要
   ツール: run_command + command_status
```

---

## パターン 1: Parallel Investigate（並列調査）

> 複数の独立した問題を同時に調査する

### いつ使う？

- 3つ以上のテストが異なる原因で失敗
- 複数のサブシステムで独立したバグ
- 新しいコードベースの複数領域を同時に理解したい

### フロー

```text
Step 1: 問題の独立性を判定
  └─ 問題Aの修正が問題Bに影響するか？ → NO = 並列可能

Step 2: 並列調査実行
  ├─ view_file × N（同時にN個のファイルを読込）
  ├─ grep_search × N（同時にN個のパターンを検索）
  └─ run_command（バックグラウンドでテスト実行しながら調査）

Step 3: 結果統合
  └─ 全調査結果を統合し、修正計画を策定
```

### 実例

```javascript
// 3ファイル同時読込（並列ツールコール）
view_file({ AbsolutePath: "/path/to/auth.test.js" })
view_file({ AbsolutePath: "/path/to/api.test.js" })
view_file({ AbsolutePath: "/path/to/db.test.js" })
// → 3つが同時に返ってくる

// バックグラウンドでテスト実行しながら調査
run_command({ CommandLine: "npx jest --verbose", WaitMsBeforeAsync: 500 })
// → テスト実行中に他のツールで調査を続行
// → command_status で結果を確認
```

---

## パターン 2: Parallel Build（並列構築）

> 独立したモジュールを同時に構築する

### P2: いつ使う？

- 複数の独立コンポーネントを実装する計画がある
- ファイル間に依存関係がない
- ultra-plan / ultra-implement 実行中

### P2: フロー

```text
Step 1: タスク依存グラフ作成
  ├─ A → B → E（直列: Bに必要なAを先に実装）
  ├─ C（独立: 並列可能）
  └─ D（独立: 並列可能）

Step 2: 独立タスクを並列実行
  ├─ write_to_file: Cのコード
  ├─ 同時に run_command: テスト実行（バックグラウンド）
  └─ ※ 同一ファイルへの並列書き込みは禁止

Step 3: 依存タスクを直列実行
  └─ A完了 → B実装 → E実装
```

### 安全ルール

```yaml
並列書き込み禁止ルール:
  - 同一ファイルへの write_to_file / replace_file_content は絶対に並列化しない
  - 同一ディレクトリへの mkdir + write も並列化しない
  - テストと実装の並列は OK（異なるファイルなので）

並列安全マトリクス:
  read × read:    ✅ 安全（何個でも並列可能）
  read × write:   ✅ 安全（異なるファイルの場合）
  write × write:  ⚠️ 異なるファイルなら安全
  write × write:  ❌ 同一ファイルは禁止
  build × test:   ✅ 安全（バックグラウンド）
  test × test:    ⚠️ ポートやDB競合に注意
```

---

## パターン 3: Browser Delegate（ブラウザ委譲）

> UI検証をサブエージェントに委譲し、メインはコード作業を継続

### P3: いつ使う？

- 実装しながら同時にUI確認したい
- E2Eテストレベルの操作検証
- スクリーンショットベースのデザイン検証

### P3: フロー

```text
Step 1: dev server をバックグラウンドで起動
  run_command: npm run dev (WaitMsBeforeAsync: 2000)

Step 2: browser_subagent に検証タスクを委譲
  browser_subagent({
    Task: "http://localhost:5173 にアクセスし、
           ログインフォームが表示されることを確認。
           スクリーンショットを撮影して返す。",
    TaskName: "Login Page Verification"
  })

Step 3: サブエージェント実行中にコード作業を継続
  → サブエージェントの結果を待ってから統合
```

### サブエージェント プロンプト設計原則

```yaml
良いプロンプト:
  - 明確なゴール: 「ログインページが表示されることを確認」
  - 具体的な手順: 「URLにアクセス → フォーム確認 → スクリーンショット」
  - 完了条件: 「スクリーンショットを撮影して返す」
  - スコープ制限: 「ログインページのみ確認、他のページには遷移しない」

悪いプロンプト:
  - 曖昧: 「サイトを確認して」
  - 広すぎ: 「全ページをテストして」
  - 完了条件なし: 「ログインを試して」
```

### 実例: 実装 + UI検証 パイプライン

```text
# 1. バックグラウンドでdev server起動
run_command: npm run dev (bg)

# 2. コード修正
replace_file_content: notes.js にMarkdownプレビュー追加

# 3. browser_subagent に検証委譲
browser_subagent:
  Task: "localhost:5173/notes にアクセスし、
         メモ追加ボタンをクリック、
         Markdownプレビュータブが表示されるか確認、
         スクリーンショット撮影"

# 4. サブエージェント結果を確認
# → 問題あれば修正 → 再度サブエージェント委譲
```

---

## パターン 4: Background Pipeline（バックグラウンドパイプライン）

> 長時間プロセスをバックグラウンドで実行しながら他作業を継続

### P4: いつ使う？

- ビルド中にドキュメント作成
- テスト実行中にコードレビュー
- デプロイ中に次タスクの準備

### P4: フロー

```text
Step 1: 長時間プロセスをバックグラウンド起動
  run_command({
    CommandLine: "npx jest --coverage",
    WaitMsBeforeAsync: 500  // すぐにバックグラウンドへ
  })
  → CommandId を記録

Step 2: バックグラウンド実行中に他作業
  ├─ ドキュメント更新
  ├─ 次のファイル読込
  └─ コードレビュー

Step 3: 定期的にステータス確認
  command_status({
    CommandId: "記録したID",
    WaitDurationSeconds: 0  // ノンブロッキング確認
  })

Step 4: 完了時に結果統合
  command_status({
    CommandId: "記録したID",
    WaitDurationSeconds: 300  // 完了まで待機
  })
```

### パイプライン例: テスト + lint + ビルド

```text
# ステージ1: 全プロセスをバックグラウンド起動
run_command: npx jest --coverage (bg) → ID_test
run_command: npx eslint src/ (bg)     → ID_lint

# ステージ2: 並行してコード作業
write_to_file: README.md 更新
view_file: 次のタスクのファイル確認

# ステージ3: 結果確認
command_status: ID_test → テスト結果
command_status: ID_lint → lint結果

# ステージ4: 結果に応じて対応
# テスト失敗 → 修正 → 再テスト（バックグラウンド）
# lint エラー → 修正
```

---

## サブエージェント駆動開発（SDD）

> ultra-implement の各タスクを browser_subagent に委譲して検証するパターン

### ワークフロー

```text
Controller（メインエージェント）
  │
  ├─ ultra-plan で計画策定
  │
  ├─ Task 1: 実装
  │    ├─ コード実装（Controller直接）
  │    ├─ browser_subagent: UI検証委譲
  │    └─ 結果確認 → 次タスクへ
  │
  ├─ Task 2: 実装
  │    ├─ コード実装（Controller直接）
  │    ├─ run_command: テスト実行（バックグラウンド）
  │    ├─ browser_subagent: UI検証委譲
  │    └─ 結果統合 → 次タスクへ
  │
  └─ 全タスク完了
       ├─ browser_subagent: 最終E2E検証
       └─ ultra-ship へ
```

### Controller の責務

```yaml
Controller がやること:
  - タスクの依存関係を分析し実行順序を決定
  - 各タスクに必要なコンテキストを整理
  - サブエージェントのプロンプトを設計
  - 結果の統合と品質判断
  - コンフリクト検出と解消

Controller がやらないこと:
  - ブラウザ操作（browser_subagent に委譲）
  - 長時間の待機（バックグラウンド実行に委譲）
```

---

## 判定フローチャート

```text
タスクが2つ以上ある？
  └─ NO → 通常の直列実行
  └─ YES
      └─ タスク間に依存関係がある？
          └─ YES → 依存順に直列、独立部分のみ並列
          └─ NO
              └─ 同一ファイルを編集する？
                  └─ YES → 直列実行（並列書き込み禁止）
                  └─ NO
                      └─ UI検証が必要？
                          ├─ YES → Pattern 3: Browser Delegate
                          └─ NO
                              └─ 長時間プロセスがある？
                                  ├─ YES → Pattern 4: Background Pipeline
                                  └─ NO → Pattern 1 or 2
```

---

## 安全ガードレール

### 絶対禁止

- 同一ファイルへの並列書き込み
- 依存関係があるタスクの並列実行
- 共有リソース（DB、ポート）への同時アクセス
- browser_subagent の複数同時起動
- バックグラウンドコマンドの放置（必ず command_status で確認）

### 並列実行前チェックリスト

```text
[ ] タスクの独立性を確認した
[ ] 同一ファイルへの並列書き込みがないことを確認した
[ ] 共有リソースの競合がないことを確認した
[ ] 各タスクの完了条件が明確である
[ ] 結果統合の手順が決まっている
```

### コンフリクト検出

並列実行後は必ず以下を確認:

```bash
# 変更ファイルの重複チェック
run_command: git diff --name-only
```

重複があれば手動で統合。

---

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-PARALLEL 完了                     ║
╠══════════════════════════════════════════╣
║  実行パターン: [Pattern名]               ║
║  並列タスク数: [N] 件                    ║
║  時間短縮率:   約 [N]x                   ║
║  コンフリクト: [N] 件                    ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╚══════════════════════════════════════════╝
```
