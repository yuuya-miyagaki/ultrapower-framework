---
name: ultra-retro
description: "スプリント/タスク完了後に起動。振り返りを実施し、学びをMemory MCPに永続化する。「振り返り」「レトロ」「何を学んだか」で起動。引数: 7d/14d/30d/24h/compare/global/auto。"
---

# Ultra Retro — レトロスペクティブ + 学習永続化

> gstack /retro + Memory MCP 永続化の統合

## 起動条件

- ultra-ship 完了後
- 「振り返りしよう」「何がうまくいった？」
- スプリント/プロジェクト区切り時

## 引数

- `/retro` — デフォルト: 7日間
- `/retro 24h` — 過去24時間
- `/retro 14d` — 過去14日間
- `/retro 30d` — 過去30日間
- `/retro compare` — 今回 vs 前回の同期間比較
- `/retro global` — 複数リポジトリ横断レトロ（7d デフォルト）
- `/retro auto` — フレームワーク自己進化（プロジェクト完了時に自動で学びをスキルに追記）

**`global` が指定された場合**: 通常の Step 1-7 をスキップし、[Global Retrospective](#global-retrospective) セクションに進む。

## Step 1: データ収集

### 1.1 時間ウィンドウ計算

引数から日数（`d`）/時間（`h`）/週（`w`）を解析。デフォルト7日。
日/週の場合は**ローカルの深夜0時**に合わせた絶対日時を使用。

```bash
run_command: git fetch origin --quiet
run_command: git config user.name
run_command: git config user.email
```

### 1.2 Gitデータ取得

以下を全て取得（`<default>` は動的に検出したデフォルトブランチ名）:

```bash
# デフォルトブランチ名を動的検出
run_command: git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main"

# コミット一覧（ハッシュ、著者、日時、件名、ファイル統計）
run_command: git log origin/<default> --since="<window>" --format="%H|%aN|%ae|%ai|%s" --shortstat

# コミット種別ごとの行数
run_command: git log origin/<default> --since="<window>" --format="COMMIT:%H|%aN" --numstat

# タイムスタンプ（セッション検出用）
run_command: git log origin/<default> --since="<window>" --format="%at|%aN|%ai|%s" | sort -n

# ホットスポット分析
run_command: git log origin/<default> --since="<window>" --format="" --name-only | grep -v '^$' | sort | uniq -c | sort -rn | head -15

# 著者ごとのコミット数
run_command: git shortlog origin/<default> --since="<window>" -sn --no-merges
```

### 1.3 品質データ

- QAレポート（`docs/qa-reports/`）
- レビュー結果（`docs/reviews/`）

## Step 2: メトリクスサマリー

```text
メトリクス              値
─────────               ──────
コミット数              [N]
コントリビューター      [N]
PRマージ数              [N]
追加行数                [N]
削除行数                [N]
Net LOC                 [N]
テスト行数              [N]
テスト比率              [N]%
アクティブ日数          [N]
検出セッション          [N]
LOC/セッション時間      [N]
テスト健全性            [N] テストファイル · [M] 追加
```

### コントリビューターリーダーボード

```text
コントリビューター   コミット   +/-           主な領域
あなた (name)             32   +2400/-300    app/
alice                     12   +800/-150     components/
```

## Step 3: コミット時間分布

ローカル時間のヒストグラム:

```text
時刻  コミット  ████████████████
 09:    4      ████
 10:    8      ████████
 ...
 22:   12      ████████████
```

分析:
- ピークタイム
- デッドゾーン
- 深夜コーディング（22時以降のクラスター）

## Step 4: ワークセッション検出

連続コミット間の **45分** ギャップでセッション区切り:

| 種別 | 基準 |
|------|------|
| ディープセッション | 50分以上 |
| ミディアムセッション | 20-50分 |
| マイクロセッション | 20分未満 |

```text
=== アクティブコーディング統計 ===
合計アクティブ時間:    [N]時間
平均セッション長:      [N]分
LOC/アクティブ時間:    [N]
```

## Step 5: コミット種別分析

Conventional Commit プレフィックス（feat/fix/refactor/test/chore/docs）で分類:

```yaml
feat:     20  (40%)  ████████████████████
fix:      27  (54%)  ███████████████████████████
refactor:  2  ( 4%)  ██
```

⚠️ fix比率が50%超 → 「レビュー段階での品質ゲートを強化検討」

## Step 6: 3カテゴリ振り返り

### 🟢 うまくいったこと（Keep）

```yaml
KEEP:
1. [具体的な成功事例 — コミットデータに基づく]
   → なぜうまくいったか: [分析]
   → 再現するには: [手順/条件]
```

### 🟡 改善すべきこと（Improve）

```yaml
IMPROVE:
1. [具体的な課題 — メトリクスに基づく]
   → 影響: [時間ロス/品質低下等]
   → 提案: [具体的な改善策]
   → 次回のアクション: [何をいつまでに]
```

### 🔴 やめるべきこと（Stop）

```yaml
STOP:
1. [具体的な悪習/問題パターン — データに基づく]
   → なぜ発生したか: [根本原因]
   → 代替手段: [何に置き換えるか]
```

## Step 7: Ultrapowerメトリクス

```yaml
ULTRAPOWER METRICS:
  デバッグループ回数:   [N] (3回制限に達した: [Y/N])
  QA-Debug サイクル数:  [N] (3ループ制限に達した: [Y/N])
  セキュリティ検知:     [N] 件
  TDD違反:             [Y/N]
```

## Step 8: ストリーク追跡

```bash
# チームストリーク
run_command: git log origin/<default> --format="%ad" --date=format:"%Y-%m-%d" | sort -u

# 個人ストリーク
run_command: git log origin/<default> --author="<user_name>" --format="%ad" --date=format:"%Y-%m-%d" | sort -u
```

今日から遡って連続コミット日数をカウント:
- 「チーム出荷ストリーク: [N]日連続」
- 「あなたの出荷ストリーク: [N]日連続」

## Step 9: 履歴ロード・比較

```yaml
find_by_name: Pattern="*.json", SearchDirectory="docs/retro-reports/"
```

見つかったファイルを日付順にソートして最新5件を取得。

前回のレトロが存在する場合、キーメトリクスの差分を表示:

```text
                    前回        今回        差分
テスト比率:         22%    →    41%         ↑19pp
セッション数:       10     →    14          ↑4
LOC/時間:           200    →    350         ↑75%
fix比率:            54%    →    30%         ↓24pp (改善)
```

前回なし → 「初回レトロ記録。次回実行でトレンドが見えます。」

## Step 10: 永続化

### JSON スナップショット保存

```bash
run_command: mkdir -p docs/retro-reports
```

`docs/retro-reports/{date}-{seq}.json` に保存。

### Memory MCP に永続化

> **重複防止:** 保存前に `mcp_memory_search_nodes` で既存エンティティ（特に ultra-debug が保存した `bug_resolution` タイプ）を検索し、重複を避ける。既存エンティティがあれば `mcp_memory_add_observations` で追記する。

```yaml
mcp_memory_create_entities:
  - name: "[プロジェクト名]-retro-[日付]"
    entityType: "retrospective"
    observations:
      - "KEEP: [学び1]"
      - "IMPROVE: [学び2]"
      - "STOP: [学び3]"
      - "PATTERN: [発見したパターン]"
      - "METRICS: コミット[N], テスト比率[N]%, ストリーク[N]d"

mcp_memory_create_relations:
  - from: "[プロジェクト名]-retro-[日付]"
    to: "[プロジェクト名]"
    relationType: "retrospective_of"
```

### アーキテクチャ決定の永続化

```yaml
mcp_memory_create_entities:
  - name: "[決定名]"
    entityType: "architecture_decision"
    observations:
      - "決定: [何を決めたか]"
      - "理由: [なぜその選択か]"
      - "代替案: [却下した案]"
      - "トレードオフ: [引き受けたリスク]"
```

## Step 11: ナラティブ出力

```yaml
一行サマリー:
Week of [date]: [N] commits, [X]k LOC, [Y]% tests | Streak: [N]d

=== Engineering Retro: [date range] ===

### サマリーテーブル（Step 2）
### トレンド比較（Step 9）
### 時間・セッションパターン（Steps 3-4）
### 出荷速度（Step 5）
### コード品質シグナル
### フォーカス＆ハイライト
### 今週のTOP 3
### 改善すべき3項目
### 来週の3つの習慣
```

## Step 12: 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-RETRO 完了                        ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  KEEP:     [N] 件                        ║
║  IMPROVE:  [N] 件                        ║
║  STOP:     [N] 件                        ║
║  ストリーク: [N]d 連続                    ║
║  学習永続化: Memory MCP に [N] 件保存     ║
║  次回アクション: [N] 件                   ║
╚══════════════════════════════════════════╝
```

---

## Global Retrospective

`/retro global` 実行時のフロー。

### Global Step 1: リポジトリ発見

```bash
# ホームディレクトリ以下のgitリポジトリを探索（デフォルトパス）
run_command: find ~/Desktop ~/Projects ~/repos ~/work -maxdepth 3 -name ".git" -type d 2>/dev/null | head -20
```

**フォールバック:** 上記のデフォルトパスでリポジトリが見つからない場合、ユーザーにプロジェクトディレクトリのパスを質問する。

**補足:** `find_by_name` はワークスペース内限定のため、ホームディレクトリ横断検索には `run_command` を使用。

### Global Step 2: 各リポのGitデータ取得

発見した各リポジトリに対して:

```bash
run_command: git -C <path> log --since="<window>" --format="%H|%aN|%ai|%s" --shortstat
run_command: git -C <path> shortlog --since="<window>" -sn --no-merges
```

### Global Step 3: グローバルストリーク

全リポの日付を統合し、連続出荷日数を計算。

### Global Step 4: コンテキストスイッチ分析

日ごとに何リポにコミットしたか:
- 平均リポ/日
- 最大リポ/日
- 集中日（1リポ）vs 分散日（3+リポ）

### Global Step 5: シェアラブルパーソナルカード

```text
╔═══════════════════════════════════════════════════════════════
║  [ユーザー名] — Week of [date]
╠═══════════════════════════════════════════════════════════════
║
║  [N] commits across [M] projects
║  +[X]k LOC added · [Y]k deleted · [Z]k net
║  [N]-day shipping streak 🔥
║
║  PROJECTS
║  ─────────────────────────────────────────────────────────
║  [repo_name]        [N] commits    +[X]k LOC    [solo/team]
║
║  TOP WORK
║  • [テーマ1]
║  • [テーマ2]
║  • [テーマ3]
║
║  Powered by Ultrapower
╚═══════════════════════════════════════════════════════════════
```

### Global Step 6: メモリ永続化 + レポート保存

JSON + Memory MCP に保存。

---

## Auto-Evolve モード (`/retro auto`)

> **フレームワーク自己進化機能**
> プロジェクト完了後に自動でフレームワーク自体を改善する。

### 起動条件

- `/retro auto` — 明示的に実行
- `ultra-ship` 完了後に自動提案（「振り返り + 自動成長を実行しますか？」）

### Auto Step 1: 変更分析

プロジェクトのgit diffからパターンを抽出:

```bash
# 今回のプロジェクトで変更されたファイルタイプと行数
run_command: git log --since="<project_start>" --format="" --numstat | awk '{ext=$3; sub(/.*\./, ".", ext); a[ext]+=$1; d[ext]+=$2} END{for(e in a) print e, a[e], d[e]}' | sort -k2 -rn

# テストファイルの変更パターン
run_command: git log --since="<project_start>" --format="%s" -- "**/*.test.*" "**/__tests__/**"

# 使用したツール/コマンドの頻度
run_command: git log --since="<project_start>" --format="%B" | grep -oE "(npm|npx|jest|vite|firebase|supabase|vercel)" | sort | uniq -c | sort -rn
```

### Auto Step 2: パターン検出

分析結果から以下のパターンを自動検出:

```yaml
検出対象:
  - 新しいテスト手法（モック方式、セットアップパターン）
  - 新しいデプロイ手順（Vercel, Firebase等）
  - 繰り返し発生したバグパターン（gotcha）
  - 新しいライブラリ/ツールの使用例
  - アダプター/抽象化パターン
  - 環境変数の落とし穴
```

### Auto Step 3: スキルマッチング

検出パターンを該当スキルにマッピング:

```yaml
マッピングルール:
  テスト関連の学び    → ultra-qa/SKILL.md
  デバッグパターン    → ultra-debug/SKILL.md
  デプロイ手順        → ultra-ship/SKILL.md
  設計パターン        → ultra-implement/SKILL.md
  レビュー観点        → ultra-review/SKILL.md
  パフォーマンス知見  → ultra-benchmark/SKILL.md
  オンボーディング情報 → ultra-onboard/SKILL.md
```

### Auto Step 4: スキル学習パターン保存

検出パターンを各スキルの **`learned-patterns.md`** に保存（SKILL.md本体は変更しない）:

```yaml
保存先:
  skills/{skill}/learned-patterns.md

保存ルール:
  1. SKILL.md は一切変更しない（肥大化防止）
  2. learned-patterns.md が存在しない場合は新規作成
  3. 既存パターンと重複がないか mcp_memory_search_nodes で照合
  4. 各スキルの learned-patterns.md は最大10件まで（超過時は古いものを統合・要約）

保存テンプレート: |
  ### [パターン名] (自動検出: [日付])

  **プロジェクト**: [PJ名]
  **状況**: [どのような場面で発生したか]
  **解決策**: [何をしたか]

  ```[言語]
  [コード例]
  ```

安全策:
  - SKILL.md の既存内容の削除・変更は一切行わない
  - 保存前にユーザーに確認プロンプトを表示
  - 保存内容を diff 表示してから適用
```

### Auto Step 5: Memory MCP 永続化

```yaml
mcp_memory_create_entities:
  - name: "auto-evolve-[日付]-[PJ名]"
    entityType: "framework_evolution"
    observations:
      - "検出パターン: [N]件"
      - "更新スキル: [スキル名1], [スキル名2]"
      - "追記内容サマリー: [概要]"

mcp_memory_create_relations:
  - from: "auto-evolve-[日付]-[PJ名]"
    to: "[PJ名]"
    relationType: "learned_from"
  - from: "auto-evolve-[日付]-[PJ名]"
    to: "Ultrapower Framework"
    relationType: "evolved"
```

### Auto Step 6: CHANGELOG 記録

```bash
run_command: cat CHANGELOG.md
```

CHANGELOG.md の先頭に進化記録を追記:

```text
## [自動進化] vX.Y.Z — YYYY-MM-DD

### 学習元: [PJ名]

- **ultra-qa**: [追記内容の1行サマリー]
- **ultra-implement**: [追記内容の1行サマリー]
- **Memory MCP**: [N]件のパターン永続化
```

### Auto Step 7: 完了レポート

```text
╔══════════════════════════════════════════╗
║  AUTO-EVOLVE 完了                        ║
╠══════════════════════════════════════════╣
║  分析したコミット:  [N] 件               ║
║  検出パターン:      [N] 件               ║
║  更新スキル:        [N] 件               ║
║  Memory MCP保存:    [N] 件               ║
║  CHANGELOG更新:     ✅                   ║
╠══════════════════════════════════════════╣
║  Ultrapower は [PJ名] から              ║
║  [N] 件の知見を吸収しました 🧠           ║
╚══════════════════════════════════════════╝
```

### 安全ガードレール

- **追記のみ**: 既存スキル内容の削除・変更は一切行わない
- **確認プロンプト**: スキル更新前に必ずユーザー承認を取る
- **ロールバック可能**: git commitで記録するため `git revert` で元に戻せる
- **重複防止**: Memory MCP検索で既存パターンと照合してから追記
