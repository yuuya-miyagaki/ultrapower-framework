# Ultrapower v4 — 自己完結型 AI 統合開発フレームワーク

> gstack × Superpowers の強みを統合。外部依存ゼロ。Antigravity + MCP ツール群を最大活用。

## クイックスタート

```bash
# 1. ultrapower/ をプロジェクトにコピー
cp -r ultrapower/ your-project/ultrapower/

# 2. 新規プロジェクトの場合
「新しいアプリを作りたい」 → ultrapower-workflow が自動でガイド

# 3. 既存プロジェクトの場合
「このコードベースを理解したい」 → ultra-onboard が起動
```

外部依存はありません。Antigravity環境のMCPツール群をそのまま活用します。

## 開発ワークフロー

```text
onboard → brainstorm → plan → implement → review → qa → ship → retro
                                  ↑                       |
                                  └──── debug ←───────────┘

独立スキル（任意タイミング）:
  benchmark / design-system / design-review / docs / second-opinion / parallel
```

### 典型的な新規PJ フロー

```text
1. 「アプリを作りたい」        → ultra-brainstorm (アイデア整理)
2. 「設計を決めよう」          → ultra-design-system (DESIGN.md生成)
3. 「実装計画を立てて」        → ultra-plan (TDD計画 + 3段階レビュー)
4. 「実装開始」                → ultra-implement (TDDサイクル実行)
5. 「テストして」              → ultra-qa (Jest + Playwright)
6. 「パフォーマンスは？」      → ultra-benchmark (バンドル/Core Web Vitals)
7. 「セキュリティチェック」    → ultra-ship (OWASP監査 + デプロイ)
8. 「振り返りしよう」          → ultra-retro (Git分析 + Memory永続化)
```

## スキル一覧（16スキル + 1オーケストレーター）

| スキル | 起動トリガー | 機能 |
| ------ | ------------ | ---- |
| **ultra-onboard** | 「コードを理解したい」 | コードベース解析、DB検出、Draw.ioアーキテクチャ図、Memory永続化 |
| **ultra-brainstorm** | 「アイデアを考えたい」 | アイデア壁打ち、設計ドキュメント生成、git init |
| **ultra-design-system** | 「デザインを決めたい」 | 競合リサーチ → SAFE/RISK提案 → DESIGN.md生成 |
| **ultra-plan** | 「実装計画を立てて」 | TDD計画 + CEO/Eng/Design 3段階レビュー + Draw.io図 |
| **ultra-implement** | 計画承認後 | TDDサイクル(RED→GREEN→REFACTOR) + 2段階レビュー |
| **ultra-review** | 実装完了時 | 検証駆動レビュー + Fix-First + 完了前検証 |
| **ultra-qa** | 「テストして」 | Jest/Playwright ブラウザQA + レスポンシブテスト |
| **ultra-benchmark** | 「パフォーマンスは？」 | Core Web Vitals + バンドルサイズ + 回帰検出 + トレンド |
| **ultra-design-review** | 「デザインを確認して」 | 7軸デザイン監査 + AIスロップ検出 + 自動修正 |
| **ultra-ship** | 「デプロイして」 | 依存監査 + OWASP/STRIDE + デプロイ(Vercel/Firebase) |
| **ultra-debug** | バグ発見時 | 4フェーズ体系的デバッグ + 並列エージェント |
| **ultra-retro** | 「振り返りしよう」 | Git分析 + セッション検出 + ストリーク + Memory永続化 |
| **ultra-docs** | 「ドキュメント更新して」 | diff分析 → 自動ドキュメント更新 → 一貫性チェック |
| **ultra-second-opinion** | 「別の意見が欲しい」 | 別AI二次意見ブリッジ + プロンプト自動生成 |
| **ultra-parallel** | 「並列で」「同時に」 | 独立タスク並列実行・browser_subagent委譲・BGパイプライン |
| **ultrapower-workflow** | 自動 | フェーズルーティング + 安全ガードレール |

## MCP ツール統合

| MCP | 使用スキル | 用途 |
| --- | ---------- | ---- |
| **Playwright** | qa, benchmark, design-review, ship | ブラウザテスト、UI検証、パフォーマンス計測 |
| **Context7** | plan, implement, debug | ライブラリ最新ドキュメント参照 |
| **Memory** | onboard, implement, retro, debug | セッション間知識永続化 |
| **Draw.io** | onboard, plan | アーキテクチャ図・フロー図の自動生成 |
| **Firebase** | ship, implement | Firebase プロジェクト管理・デプロイ |
| **GitHub** | ship, docs | PR作成、ブランチ管理 |
| **Hugging Face** | implement, brainstorm | AIモデル推論・Models Hub・Datasets Hub・Spaces活用 |
| **Filesystem** | 全スキル | ファイル読み書き・検索 |

## DB バックエンド

プロジェクトに応じて自動検出:

| シグナル | 判定 |
| -------- | ---- |
| `supabase/` dir, `SUPABASE_URL` | **Supabase** |
| `firebase.json`, `FIREBASE_` prefix | **Firebase** |
| 両方検出 / 検出なし | ユーザーに質問 |
| いずれも無し + DB不要 | **None**（DB関連スキップ） |

### Supabase 自動化

```bash
mkdir -p supabase/migrations
# マイグレーションファイル管理
npx supabase migration new init     # 作成
npx supabase db push                 # 適用
```

### Firebase 自動化

```bash
# Firebase MCP ツールでセットアップ
mcp_firebase-mcp-server_firebase_init:
  features:
    firestore: { location_id: "asia-northeast1" }
    auth: { providers: { emailPassword: true } }
```

## 実践結果

> 詳細は [docs/case-studies/](docs/case-studies/) を参照。

| プロジェクト | 実装時間 | テスト | 概要 |
| ------------ | -------- | ------ | ---- |
| StudyFlow | 74分 | 83件 100% PASS | 個人ナレッジハブ（Ultrapower v4.1で構築） |

## v4 新機能

- 📊 **ultra-benchmark**: Core Web Vitals + バンドルサイズ + 回帰検出
- 🎨 **ultra-design-system**: 競合リサーチ → DESIGN.md生成
- 🎯 **ultra-design-review**: 7軸デザイン監査 + 自動修正
- 📝 **ultra-docs**: diff分析 → 自動ドキュメント更新
- 🤝 **ultra-second-opinion**: 別AI二次意見ブリッジ
- ⚡ **ultra-parallel**: 独立タスク並列実行 + サブエージェント委譲
- 🔄 **ultra-retro強化**: セッション検出 + ストリーク + global横断
- 🤖 **ultra-implement強化**: 2段階レビュー + DB自動検出
- ✅ **ultra-review強化**: 検証なき完了宣言禁止

## 要件

- **Antigravity エージェント**（Google DeepMind 提供）
- 以下の MCP サーバーが利用可能であること:
  - Playwright MCP（ブラウザテスト・UI検証用）
  - Context7 MCP（ライブラリドキュメント参照用）
  - Memory MCP（知識永続化用）
  - Draw.io MCP（アーキテクチャ図生成用）
  - GitHub MCP（PR・ブランチ管理用）
  - Firebase MCP（Firebase プロジェクト管理用、オプション）
- Hugging Face Pro アカウント + `huggingface_hub`（AIモデル推論・データセット・Spaces活用、オプション）
- 外部AI（Claude Code, ChatGPT 等 — ultra-second-opinion 用、オプション）

## ライセンス

プライベートフレームワーク。個人利用。
