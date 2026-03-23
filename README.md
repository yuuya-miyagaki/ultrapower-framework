# Ultrapower v4 — 自己完結型 AI 統合開発フレームワーク

> gstack × Superpowers の強みを統合。外部依存ゼロ。Antigravity + MCP ツール群を最大活用。

## インストール

```
ultrapower/ を任意のプロジェクトにコピー
```

外部依存はありません。Antigravity環境のMCPツール群をそのまま活用します。

## スキル一覧（14スキル + 1オーケストレーター）

| スキル | 機能 |
| ------ | ---- |
| **ultra-onboard** | コードベース理解、DB検出、アーキテクチャ図（Draw.io）、Context7ドキュメント、Memory永続化 |
| **ultra-brainstorm** | アイデア壁打ち、設計ドキュメント生成 |
| **ultra-plan** | TDD実装計画 + CEO/Eng/Design 3段階レビュー、Context7検証、Draw.ioアーキテクチャ図 |
| **ultra-implement** | TDDサイクル実行 + サブエージェント駆動開発、Context7参照、DB自動検出、Memory永続化 |
| **ultra-review** | 検証駆動コードレビュー + Fix-First + 受理プロトコル + 完了前検証 |
| **ultra-qa** | Playwright MCP ブラウザQA + レスポンシブテスト |
| **ultra-ship** | 依存関係監査 + OWASP/STRIDE セキュリティ監査 + 4オプション完了フロー + DB対応デプロイ |
| **ultra-debug** | 4フェーズ体系的デバッグ + 並列エージェント + Context7/Memory活用 |
| **ultra-retro** | Git分析レトロ + セッション検出 + ストリーク + global横断 + Memory永続化 |
| **ultra-benchmark** | パフォーマンス計測 + Core Web Vitals + バンドルサイズ + 回帰検出 + トレンド分析 |
| **ultra-design-system** | デザインシステム構築 + 競合リサーチ + SAFE/RISK提案 + DESIGN.md生成 |
| **ultra-design-review** | デザイン監査 + 7軸診断 + AIスロップ検出 + 自動修正 + before/afterスクリーンショット |
| **ultra-docs** | ドキュメント同期 + 自動更新 + CHANGELOG保護 + 一貫性チェック |
| **ultra-second-opinion** | Claude Code二次意見ブリッジ + コンテキスト自動収集 + コピペ用プロンプト生成 |
| **ultrapower-workflow** | 自動フェーズルーティング + 安全ガードレール |

## ワークフロー

```
onboard → brainstorm → plan → implement → review → qa → ship → retro
                                  ↑                       |
                                  └──── debug ←───────────┘

独立スキル（任意タイミング）:
  benchmark / design-system / design-review / docs / second-opinion
```

## MCP ツール活用

| MCP | 用途 |
| --- | ---- |
| **Playwright** | ブラウザテスト、UI QA、デザイン監査、パフォーマンス計測、デプロイ確認 |
| **Context7** | ライブラリ最新ドキュメント参照 |
| **Memory** | セッション間知識永続化 |
| **Draw.io** | アーキテクチャ図・フロー図の自動生成 |
| **Filesystem** | ファイル読み書き・ディレクトリ操作・検索 |
| **Firebase** | Firebase プロジェクトのデプロイ・管理 |
| **GitHub** | PR作成、ブランチ管理、ドキュメント連携 |

## DB バックエンド

プロジェクトに応じて自動検出:
- **Supabase**: `supabase/` ディレクトリ or `SUPABASE_URL` を検出
- **Firebase**: `firebase.json` or `FIREBASE_` 環境変数を検出
- **その他**: ユーザーに質問

## v4 新機能

- 📊 **ultra-benchmark**: Playwright MCPでCore Web Vitals計測、回帰検出、トレンド分析
- 🎨 **ultra-design-system**: 競合リサーチ → 統合提案 → DESIGN.md生成、AIスロップ検出
- 🎯 **ultra-design-review**: ライブサイト7軸デザイン監査 + 自動修正 + before/afterスクリーンショット
- 📝 **ultra-docs**: diff分析 → 自動ドキュメント更新 → 一貫性チェック
- 🤝 **ultra-second-opinion**: Claude Codeへの二次意見ブリッジ、コピペ用プロンプト自動生成
- 🔄 **ultra-retro強化**: Gitセッション検出、ストリーク追跡、compare、globalモード
- 🤖 **ultra-implement強化**: サブエージェント駆動開発パターン、2段階レビュー
- ✅ **ultra-review強化**: 「検証なき完了宣言禁止」の鉄則、合理化防止テーブル

## 要件

- **Antigravity エージェント**（Google DeepMind 提供）
- 以下の MCP サーバーが利用可能であること:
  - Playwright MCP（ブラウザテスト・UI検証用）
  - Context7 MCP（ライブラリドキュメント参照用）
  - Memory MCP（知識永続化用）
  - Draw.io MCP（アーキテクチャ図生成用）
  - GitHub MCP（PR・ブランチ管理用）
  - Firebase MCP（Firebase プロジェクト管理用、オプション）
- Claude Code（ultra-second-opinion 用、オプション）
