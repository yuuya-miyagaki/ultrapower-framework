---
name: ultrapower-workflow
description: "常時アクティブ。ユーザーの入力とコンテキストから適切なフェーズスキルを自動選択・起動するオーケストレーター。"
---

# Ultrapower Workflow — 自動フェーズオーケストレーター

ユーザーの入力と現在のプロジェクト状態から、適切なUltrapower スキルを自動で起動する。

> **自己完結型** — 外部フレームワーク不要。Antigravity MCP ツール群を活用。

## 自動ルーティング

| ユーザーの入力/状態 | 起動スキル |
| -------------------- | ---------- |
| 新プロジェクト・「コード読んで」 | ultra-onboard |
| 「作りたい」「アイデアがある」「新機能」 | ultra-brainstorm |
| 設計が承認された | ultra-plan |
| 計画が承認された | ultra-implement |
| 「実装完了」「レビューして」 | ultra-review |
| 「テストして」「QA」、レビュー通過 | ultra-qa |
| 「出荷」「PRを作って」「デプロイ」、QA通過 | ultra-ship |
| エラーが発生、テスト失敗、「バグ」 | ultra-debug |
| 「振り返り」「レトロ」、ship完了後 | ultra-retro |
| 「パフォーマンス」「ベンチマーク」「ページ速度」 | ultra-benchmark |
| 「デザインシステム」「ブランド」「DESIGN.md」 | ultra-design-system |
| 「デザインチェック」「見た目」「デザイン監査」 | ultra-design-review |
| 「ドキュメント更新」「docs同期」、PRマージ後 | ultra-docs |
| 「二次意見」「セカンドオピニオン」「別の目で」 | ultra-second-opinion |

## フェーズ遷移ルール

```
onboard → brainstorm → plan → implement → review → qa → ship → retro
                                  ↑                       |
                                  └──── debug ←───────────┘
                                  ↑
                                  └──── qa (バグ発見時)

独立スキル（任意のタイミングで起動可能）:
  ├── ultra-benchmark      ← パフォーマンス計測
  ├── ultra-design-system  ← デザインシステム構築
  ├── ultra-design-review  ← デザイン監査＋修正
  ├── ultra-docs           ← ドキュメント同期
  └── ultra-second-opinion ← Claude Code二次意見
```

### 自動遷移

1. **ultra-onboard** → 理解完了 → **ultra-brainstorm** (新規開発時)
   - **新規プロジェクト（コードベースが存在しない場合）**: ultra-onboard をスキップし、直接 **ultra-brainstorm** に遷移。ultra-onboard は既存コードの理解に特化しており、新規PJでは大半のステップがスキップされるため。
2. **ultra-brainstorm** → 設計承認 → **ultra-plan**
3. **ultra-plan** → 計画承認 → **ultra-implement**
4. **ultra-implement** → 全タスク完了 → **ultra-review**
5. **ultra-review** → 全指摘解決 → **ultra-qa**
6. **ultra-qa** → 全チェック通過 → **ultra-ship**
7. **ultra-ship** → 出荷完了 → **ultra-retro**（推奨）+ **ultra-docs**（推奨）
8. **ultra-retro** → 振り返り完了（終端）

### 推奨遷移（自動ではないが提案）

- **新規プロジェクト開始時** → ultra-design-system を提案
- **PR作成前** → ultra-benchmark を提案（Webアプリの場合）
- **ultra-qa完了後** → ultra-design-review を提案（Webアプリの場合）
- **ultra-review完了後** → ultra-second-opinion を提案（重要な変更の場合）
- **ship完了後** → ultra-docs を提案

### 独立スキルのフィードバック機構

独立スキルの結果は以下のようにパイプラインにフィードバックする:

| 独立スキル | フィードバック先 | トリガー条件 |
|-------------|----------------|------------------|
| ultra-benchmark | ultra-implement | REGRESSION検出時 → パフォーマンス改善タスクを実装計画に追加 |
| ultra-design-review | ultra-implement | 修正が必要な問題検出時 → CSS修正タスクを実装計画に追加 |
| ultra-design-system | ultra-brainstorm | DESIGN.md作成時 → 次回のbrainstormで参照 |
| ultra-docs | 終端 | ドキュメント更新のみ。パイプラインには影響なし |
| ultra-second-opinion | ultra-plan / ultra-implement | Plan Reviewモード → ultra-plan修正。Review/Challengeモード → ultra-implement修正 |

### 分岐

- **バグ発見**（どのフェーズでも）→ **ultra-debug**
- **ultra-debug** 完了 → **ultra-review**（修正のレビュー）
- **QA** でバグ → **ultra-debug** → 修正 → **QA** 再実行（最大3ループ）

## 直接起動

ユーザーが直接スキルを指定した場合は、ルーティングをバイパス：

- `/onboard` → ultra-onboard
- `/brainstorm` → ultra-brainstorm
- `/plan` → ultra-plan
- `/implement` → ultra-implement
- `/review` → ultra-review
- `/qa` → ultra-qa
- `/ship` → ultra-ship
- `/debug` → ultra-debug
- `/retro` → ultra-retro
- `/benchmark` → ultra-benchmark
- `/design-system` → ultra-design-system
- `/design-review` → ultra-design-review
- `/docs` → ultra-docs
- `/second-opinion` → ultra-second-opinion

## 絶対ルール（AGENTS.md 「5つの鉄則」+ 「安全ガードレール」参照）

AGENTS.md の鉄則を全スキルで厳守。特に:
- **テストファースト** — 失敗するテストなしに本番コードを書かない
- **検証なし完了宣言禁止** — 証拠が先、主張は後
- **3回失敗でエスカレーション** — 同じ問題に3回以上挑戦しない

## ブラウザ操作

Playwright MCP を使用（AGENTS.md のマッピング表参照）。

## DB バックエンド

プロジェクトに応じて Supabase / Firebase を自動検出（AGENTS.md 参照）。
