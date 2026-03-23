---
name: ultra-review
description: "実装完了後に起動。検証なき完了宣言禁止 + Fix-Firstの2パスコードレビュー + レビュー受理プロトコル + 品質ゲート。「レビューして」「コードチェック」で起動。"
---

# Ultra Review — 検証駆動コードレビュー

> gstack review + Superpowers receiving-code-review + verification-before-completion の統合

## 起動条件

- 「レビューして」「コードを見て」と言われた時
- ultra-implement 完了後
- ultra-debug 完了後（修正のレビュー）

---

## Step 1: 鉄則 — 検証なき完了宣言禁止

> AGENTS.md「5つの鉄則」に準拠。詳細はAGENTS.mdを参照。

### レビュー固有の検証パターン

| 主張 | 必要な証拠 | 不十分な証拠 |
|------|-----------|-------------|
| テスト通過 | テストコマンド出力: 0 failures | 前回の実行、「通るはず」 |
| ビルド成功 | ビルドコマンド: exit 0 | リンター通過 |
| バグ修正 | 元の症状テスト: pass | コード変更、修正したと仮定 |
| 要件充足 | 行ごとのチェックリスト | テストが通る |

---

## Step 2: レビュー前準備

```bash
# デフォルトブランチ名を動的検出
run_command: git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@' || echo "main"

run_command: git diff <default>...HEAD --stat
run_command: git log <default>..HEAD --oneline
```

`<default>` は上記で検出したデフォルトブランチ名。変更の全体像を把握し、タッチされたファイルを全て読む。

## Step 3: 2パスレビュー

### Pass 1: 仕様準拠チェック（Spec Compliance）

計画書・仕様書との整合性を最優先で検証:

| チェック項目 | 検証方法 |
|------------|---------|
| 計画の全タスクが実装されている | 計画書チェックリストとの突合 |
| 要件の解釈が正しい | 仕様の各条件をコードで確認 |
| TDD サイクルが守られている | テスト → 実装の順序をコミット履歴で確認 |
| スコープ外の実装がない（YAGNI） | 計画にない機能が追加されていないか |
| エッジケースが網羅されている | 仕様記載のエッジケース vs テスト |

```
Pass 1 判定:
  ✅ PASS → Pass 2 に進む
  ❌ FAIL → 不一致箇所をリスト化し、修正してから再レビュー
```

### Pass 2: コード品質チェック（Code Quality）

#### CRITICAL（必ず修正）

以下のパターンを検出したら即座に修正:

| パターン | 対応 |
| -------- | ---- |
| SQL文字列結合 | パラメータ化クエリに **AUTO-FIX** |
| ユーザー入力の未サニタイズ | バリデーション追加 **AUTO-FIX** |
| レースコンディション | ロック/アトミック操作 **ASK** |
| ハードコードされたシークレット | 環境変数に **AUTO-FIX** |
| LLM出力の無検証利用 | バリデーション追加 **AUTO-FIX** |
| N+1クエリ | Eager loading **AUTO-FIX** |
| デッドコード | 削除 **AUTO-FIX** |
| 古いコメント | 更新/削除 **AUTO-FIX** |

**AUTO-FIX**: 質問せずに即座に修正してコミット
**ASK**: ユーザーに確認してから修正

#### INFORMATIONAL（報告のみ）

- コードスタイルの一貫性
- 命名改善の提案
- リファクタリング機会
- ドキュメント改善

報告のみ。修正はユーザーが決定。

## Step 4: レビュー受理プロトコル（Superpowers統合）

### 外部レビューを受けた場合の対応

```
フィードバック受信時:
1. READ: 全フィードバックを反応せずに読む
2. UNDERSTAND: 要件を自分の言葉で再述（or 質問）
3. VERIFY: コードベースの現実と照合
4. EVALUATE: このコードベースにとって技術的に正しいか？
5. RESPOND: 技術的確認 or 理由付きプッシュバック
6. IMPLEMENT: 1つずつ実装、各々テスト
```

### 禁止レスポンス

- ❌ 「You're absolutely right!」（パフォーマティブ）
- ❌ 「Great point!」「Excellent feedback!」
- ❌ 「Let me implement that now」（検証前）

### YAGNI チェック

```
IF reviewer suggests "implementing properly":
  → grep codebase for actual usage
  → IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  → IF used: Then implement properly
```

### プッシュバック基準

以下の場合はプッシュバック:
- 既存機能を壊す提案
- レビュアーがフルコンテキストを把握していない
- YAGNI違反（未使用機能の追加）
- このスタックでは技術的に不正確
- ユーザーのアーキテクチャ決定と矛盾

### 実装順序

```
1. 不明点を全て先に確認
2. 実装順序:
   - ブロッキング問題（破壊・セキュリティ）
   - 単純修正（タイポ・インポート）
   - 複雑修正（リファクタリング・ロジック）
3. 各修正を個別にテスト
4. リグレッションなしを確認
```

## Step 5: 品質ゲート

全条件が満たされるまで PASS しない:

- [ ] テストが全て通過（`run_command` で**このメッセージ内で**実行確認済み）
- [ ] セキュリティパターン違反ゼロ
- [ ] 計画の全タスクが実装されている
- [ ] TDD サイクルが守られている（テスト → 実装の順）
- [ ] 自動修正可能な問題は全て修正済み
- [ ] **フレッシュな検証証拠**が全ての合格主張に存在する

## Step 6: 完了レポート

```
╔══════════════════════════════════════════╗
║  ULTRA-REVIEW 完了                       ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  CRITICAL 検出:  [N] 件                  ║
║    AUTO-FIX:     [N] 件                  ║
║    ASK:          [N] 件                  ║
║  INFORMATIONAL:  [N] 件                  ║
║  検証証拠:       全主張に [Y/N]           ║
║  品質ゲート:     [PASS/FAIL]             ║
║  次のフェーズ:   ultra-qa                ║
╚══════════════════════════════════════════╝
```

品質ゲート PASS → ultra-qa を推奨
品質ゲート FAIL → 問題リストと修正を提案
