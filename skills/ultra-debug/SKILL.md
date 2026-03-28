---
name: ultra-debug
description: "エラー、テスト失敗、バグ発生時に起動。4ステップ体系的デバッグ + 並列エージェント派遣。根本原因を理解してから修正する。"
---

# Ultra Debug — 4ステップ体系的デバッグ + 並列調査

> gstack investigate + Superpowers systematic-debugging + dispatching-parallel-agents の統合

## 起動条件

- エラーが発生した時
- テストが失敗した時
- 「バグ」「デバッグ」「調査して」と言われた時
- ultra-qa でバグが発見された時

> **学習パターン参照**: 同ディレクトリの `learned-patterns.md` が存在する場合、スキル実行時に参照し、過去の知見を活用する。

## 前提ルール（AGENTS.md「5つの鉄則」参照）

1. **根本原因を理解するまで修正コードを書かない**
2. **推測で修正しない** — 仮説を立て、検証してから修正
3. **修正前に失敗するテストを書く**（TDD厳守）
4. **3回失敗したらエスカレーション**

---

## Step 0: Freeze ゾーン宣言（スコープ外変更防止）

> **「デバッグ中に無関係なファイルを変更してバグを拡大する」ことを防ぐ。**
> 修正対象のファイルを明示的に宣言し、それ以外のファイルへの変更を禁止する。

### 0.1 影響範囲の初期スキャン

```bash
# エラーが発生したファイルを特定
run_command: grep -rn "[エラーメッセージ]" src/ --include="*.{js,ts,jsx,tsx,py}" | head -10

# エラーファイルの import 依存を確認
run_command: grep -E "^(import|from|require)" [エラーファイル] | head -20
```

### 0.2 Freeze ゾーン宣言

```yaml
FREEZE_ZONE:
  editable:
    - "[修正対象ファイル1]"
    - "[修正対象ファイル2]"
    - "[関連テストファイル]"
  frozen:
    - "上記以外の全ファイル"
  reason: "[なぜこのスコープか]"
```

### 0.3 Freeze ルール

| ルール | 内容 |
| ------ | ---- |
| **変更禁止** | `frozen` リストのファイルは読み取り専用。編集しない |
| **スコープ拡大** | 調査の結果、追加ファイルの変更が必要な場合は、ユーザーに承認を求めてから `editable` に追加 |
| **テストファイル** | 修正対象のテストファイルは自動的に `editable` に含まれる |
| **設定ファイル** | `.env`, `config.*` 等の設定ファイルは原則 `frozen`。変更が必要な場合は明示的に宣言 |

### 0.4 Freeze 違反チェック

修正実装後（Step 4 完了時）に自動チェック:

```bash
# 変更されたファイル一覧を取得
run_command: git diff --name-only HEAD~1 2>/dev/null || echo "git not available"

# editable リストとの照合
# editable に含まれないファイルが変更されていたら FREEZE_VIOLATION を報告
```

```text
⚠️ FREEZE VIOLATION DETECTED:
  変更されたファイル: [ファイル名]
  Freeze ゾーン: frozen
  対応: 変更を revert するか、ユーザーに承認を求める
```

---

## Step 1: 根本原因調査

### 1.1 症状の正確な記録

```yaml
SYMPTOM:       [何が起きているか]
EXPECTED:      [何が起きるべきか]
REPRODUCIBLE:  [再現手順]
FREQUENCY:     [常時/断続的/特定条件]
```

### 1.2 情報収集

```bash
# エラーログ確認
run_command: [テスト実行コマンド]

# 関連コードの検索
grep_search: [エラーメッセージ]

# 最近の変更確認
run_command: git log --oneline -10
run_command: git diff HEAD~3 --stat
```

### 1.3 Memory MCP で過去の知識検索

```yaml
mcp_memory_search_nodes: "[エラーパターン or 関連機能名]"
```

過去に同様の問題を解決した記録があれば、その知識を活用。

### 1.4 Context7 でライブラリの既知問題検索

```yaml
mcp_context7_resolve-library-id: [関連ライブラリ]
mcp_context7_query-docs: "[エラーメッセージ]の原因と解決方法"
```

### 1.5 ブラウザ問題の場合

> **ツール選択**: Playwright MCP が利用可能なら使用。未搭載の場合は `browser_subagent` で代替（AGENTS.md 参照）。

#### Playwright MCP 使用時

```text
mcp_playwright_browser_navigate → 問題のURL
mcp_playwright_browser_console_messages → エラー確認
mcp_playwright_browser_network_requests → API失敗確認
mcp_playwright_browser_snapshot → DOM状態確認
```

#### browser_subagent 使用時（フォールバック）

```yaml
browser_subagent:
  Task: |
    1. [問題のURL] にアクセス
    2. コンソールエラーを確認して報告
    3. ネットワークタブで失敗リクエストを確認して報告
    4. DOM状態のスクリーンショットを撮影
  RecordingName: debug_browser
```

## Step 2: パターン分析

### 2.1 問題の分類

| パターン | 調査方向 |
| -------- | -------- |
| 状態管理バグ | 状態遷移を追跡、競合状態を確認 |
| データフローバグ | 入力→変換→出力を追跡 |
| 統合バグ | API境界、型の不一致 |
| タイミングバグ | 非同期処理、レースコンディション |
| 環境バグ | 設定、依存関係、パス |

### 2.2 影響範囲の特定

```yaml
AFFECTED FILES:    [影響を受けるファイル]
AFFECTED TESTS:    [影響を受けるテスト]
RELATED SYSTEMS:   [関連するシステム]
BLAST RADIUS:      [影響範囲の広さ: 小/中/大]
```

## Step 3: 仮説と検証

### 3.1 仮説の構築

```yaml
HYPOTHESIS 1: [仮説]
  EVIDENCE FOR:     [支持する証拠]
  EVIDENCE AGAINST: [反証]
  TEST:             [検証方法]

HYPOTHESIS 2: [仮説]
  ...
```

### 3.2 検証実行

各仮説について:
1. 検証コマンドを実行（`run_command`）
2. 結果を記録
3. 仮説を確認/否定

**3回検証しても原因特定できない場合 → エスカレーション**

## Step 4: 修正実装

### 4.1 テストファースト

```text
1. 失敗するテストを書く（根本原因を再現）
2. テストが失敗することを確認（RED）
3. 最小限の修正を実装
4. テストが通過することを確認（GREEN）
5. リファクタリング（必要なら）
6. 全テストスイートを実行（リグレッション確認）
```

### 4.2 修正のコミット

```bash
git add [修正ファイル]
git commit -m "fix: [バグの簡潔な説明]

Root cause: [根本原因]
Test: [追加したテスト]"
```

## 複数問題の分離調査戦略

### 分離調査の条件

- 3つ以上のテストファイルが異なる原因で失敗
- 複数のサブシステムが独立して壊れている
- 各問題が他の問題のコンテキストなしに理解可能

### 分離しない条件

- 失敗が関連している（1つ直せば他も直る可能性）
- システム全体の状態理解が必要

### 逐次集中調査パターン

```text
1. 独立した問題ドメインを特定
   - File A テスト: ツール承認フロー
   - File B テスト: バッチ完了動作
   - File C テスト: アボート機能

2. 各ドメインを1つずつ集中的に調査・修正
   - 具体的スコープ: 1テストファイル/サブシステム
   - 明確な目標: このテストを通す
   - 制約: 他のコードは変更しない
   - 修正後: 全テストスイート実行でリグレッション確認

3. 全ドメイン完了後の統合検証
   - 修正が競合していないか確認
   - 全テストスイート実行
```

### ブラウザ問題の並行調査（browser_subagent）

UIに関連するバグの場合のみ `browser_subagent` を使用:

```yaml
browser_subagent:
  Task: |
    以下のURLにアクセスし、問題を調査してください。
    URL: [問題のURL]
    調査観点:
    - コンソールエラーの有無
    - ネットワークリクエストの失敗
    - DOM状態の確認
    - スクリーンショット取得
    完了したら調査結果を報告してください。
```

**注意**: `browser_subagent` はブラウザ操作専用。コードの読み取り・編集・テスト実行には使用不可。

## 3回失敗エスカレーションプロトコル

```yaml
IF 同じ問題に3回修正を試みて失敗:
  1. 全試行の記録を整理
  2. 以下をユーザーに報告:
     - 試行した3つのアプローチ
     - 各アプローチが失敗した理由
     - 根本原因の現在の理解
  3. 選択肢を提示:
     A. アーキテクチャレビューを実施（設計自体に問題がある可能性）
     B. 別のアプローチで4回目の試行を許可
     C. この問題を保留し、他のタスクを先に進める
  4. ユーザーの選択を待つ
```

## デバッグ知識の永続化（Memory MCP）

解決したバグの知識を保存（同じ問題の再発防止）:

```yaml
mcp_memory_create_entities:
  - name: "[プロジェクト名]-bug-[バグIDまたは簡潔な説明]"
    entityType: "bug_resolution"
    observations:
      - "症状: [何が起きたか]"
      - "根本原因: [なぜ起きたか]"
      - "修正: [何をしたか]"
      - "教訓: [次回以降の注意点]"
```

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-DEBUG 完了                        ║
║  STATUS: [DONE / BLOCKED]                ║
╠══════════════════════════════════════════╣
║  根本原因:    [特定された原因]            ║
║  修正:        [実施した修正]              ║
║  追加テスト:  [N] 件                      ║
║  リグレッション: [NONE/FOUND]             ║
║  Memory保存: [保存済/N/A]              ║
║  次のフェーズ: ultra-review               ║
╚══════════════════════════════════════════╝
```
