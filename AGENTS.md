# Ultrapower — 自己完結型 AI 開発フレームワーク v4（Antigravity環境対応）

> gstack × Superpowers の統合。外部依存ゼロ。**Antigravity エージェント + MCP ツール群を最大活用。**

---

## スキル一覧

| スキル | トリガー条件 |
| ------ | ------------ |
| ultra-onboard | 新プロジェクト・「コード読んで」・初回参加 |
| ultra-brainstorm | 新機能・アイデア・「作りたい」 |
| ultra-plan | 設計が承認された時 |
| ultra-implement | 計画が承認された時 |
| ultra-review | 実装完了・「レビューして」 |
| ultra-qa | テスト要求・レビュー通過 |
| ultra-ship | 出荷・PR作成・デプロイ |
| ultra-debug | エラー・テスト失敗・バグ |
| ultra-retro | スプリント完了・振り返り |
| ultra-benchmark | パフォーマンス計測・回帰検出 |
| ultra-design-system | デザインシステム構築・提案 |
| ultra-design-review | デザイン監査・自動修正 |
| ultra-docs | ドキュメント同期・整合性チェック |
| ultra-second-opinion | Claude Code二次意見ブリッジ |
| ultrapower-workflow | 常時（自動ルーティング） |

**1%でも関連するスキルがあればトリガーすること。**

---

## 5つの鉄則

### 1. テストファースト（TDD厳守）
失敗するテストを書いてから本番コードを書く。例外なし。

### 2. 検証なし完了宣言禁止
コマンド出力なしに「動作する」「テスト通過」と言ってはならない。
「should」「probably」「seems to」は禁止ワード。

### 3. 完全性原則（Boil the Lake）
AIで限界コストが低い→完全版を常に選ぶ。ショートカットより完全なテスト・エラー処理・エッジケース対応。

**判断基準:**
- 80行と150行の差は AI では無意味 → 常に完全版を推奨
- テストカバレッジ、エラーハンドリング、ドキュメントは「最後の10%」を省略しない
- **Lake（実行可能）**: モジュールの100%テスト、全エッジケース対応、完全なエラーパス → **やる**
- **Ocean（実行不可能）**: システム全体の書き直し、制御外の依存への機能追加 → **スコープ外と明示**

**アンチパターン:**
- ❌「90%のカバーで十分」→ 残り10%は数分で完了する
- ❌「フォローアップPRでテスト追加」→ テストは最もコストが低い Lake
- ❌「エッジケースは後で」→ エッジケース対応は AI で数分

### 4. 探索先行（Search Before Building）
車輪の再発明を避ける。3層検索:
- **Layer 1**: 定番アプローチ
- **Layer 2**: 最新のベストプラクティス
- **Layer 3**: 一般通念が間違っている可能性の検討

### 5. 3回失敗でエスカレーション
同じ問題に3回以上挑戦しない。3回失敗→ユーザーにエスカレーション。

---

## 完了ステータスプロトコル

全スキルの完了レポートで以下のステータスコードを使用:

| ステータス | 意味 | 使用条件 |
|-----------|------|---------|
| **DONE** | 全ステップ完了、証拠提示済み | 全チェック PASS |
| **DONE_WITH_CONCERNS** | 完了したが注意事項あり | 軽微な問題が残存 |
| **BLOCKED** | 続行不可能 | 外部要因で停止 |
| **NEEDS_CONTEXT** | 情報不足で判断不能 | ユーザー入力が必要 |

```
╔══════════════════════════════════════════╗
║  STATUS: [DONE / DONE_WITH_CONCERNS /   ║
║          BLOCKED / NEEDS_CONTEXT]        ║
║  REASON: [1-2行の説明]                   ║
║  EVIDENCE: [検証コマンド出力 or リンク]   ║
╚══════════════════════════════════════════╝
```

**ルール**: DONE 以外のステータスでは必ず REASON を記載すること。

---

## 安全ガードレール（gstack /careful + /freeze 統合）

### 破壊的コマンド検出

以下のパターンを検出したら**実行前に必ず警告**:

| パターン | リスク | 対応 |
| -------- | ------ | ---- |
| `rm -rf` / `rmdir` | ファイル削除 | 対象パスを確認。`/` や `~` を含む場合は**絶対停止** |
| `DROP TABLE` / `DELETE FROM` (WHERE句なし) | データ損失 | 必ずバックアップ確認 |
| `git push --force` | 履歴破壊 | `--force-with-lease` への変更を提案 |
| `git reset --hard` | コミット消失 | stash 推奨 + 確認 |
| `chmod 777` | セキュリティ | 最小権限に修正 |
| `npm publish` / `cargo publish` | 公開 | バージョン + 内容確認 |
| `firebase deploy` / `vercel deploy` | 本番影響 | ステージング確認 |
| migration (destructive) | スキーマ破壊 | ロールバック手順確認 |

### 警告フォーマット

```
⚠️ 破壊的操作を検出:
  コマンド: [検出されたコマンド]
  リスク:   [何が起きうるか]
  推奨:     [より安全な代替案]

実行しますか？ (y/N)
```

**ユーザーの明示的承認なしに破壊的コマンドを実行しない。**

---

## See Something, Say Something

スキル実行中に**現在のタスク範囲外の問題**を発見した場合:

- テスト失敗、非推奨警告、セキュリティ勧告、リンティングエラー、デッドコード、環境問題 等

**対応ルール:**
1. 問題を**1行で簡潔に報告**（何を発見し、どんな影響があるか）
2. 「修正しますか？」とユーザーに確認
3. **暗黙的に無視しない** — 発見した問題を黙って見過ごすのは禁止

```
例: 「⚡ テスト実行中に deprecation warning を3件検出しました（React 19 対応が必要）。今修正しますか？」
```

---

## DB バックエンド選択（Supabase / Firebase）

プロジェクトのDBバックエンドを自動検出し、適切なツールを使用:

### 自動検出ルール

| シグナル | 判定 |
| -------- | ---- |
| `supabase/` dir, `.env` に `SUPABASE_URL` | **Supabase** |
| `firebase.json`, `.env` に `FIREBASE_` prefix | **Firebase** |
| `prisma/schema.prisma` | **Prisma ORM** (DB種別はschema参照) |
| 両方検出 / 検出なし | **ユーザーに質問** |

### Supabase プロジェクト

```
DB操作:      Supabase Dashboard or supabase CLI
認証:        Supabase Auth (@supabase/supabase-js)
ストレージ:  Supabase Storage
マイグレーション: supabase/migrations/
デプロイ:    Vercel / Cloudflare (フロント) + Supabase (バックエンド)
```

### Firebase プロジェクト

```
DB操作:      mcp_firebase-mcp-server_* ツール群
認証:        Firebase Auth (mcp_firebase-mcp-server_firebase_init → auth)
ストレージ:  Firebase Storage
マイグレーション: Firestore Security Rules
デプロイ:    mcp_firebase-mcp-server_firebase_init → hosting
```

### Firebase MCP ツール一覧

| 操作 | ツール |
| ---- | ------ |
| プロジェクト一覧 | `mcp_firebase-mcp-server_firebase_list_projects` |
| プロジェクト情報 | `mcp_firebase-mcp-server_firebase_get_project` |
| アプリ作成 | `mcp_firebase-mcp-server_firebase_create_app` |
| SDK設定取得 | `mcp_firebase-mcp-server_firebase_get_sdk_config` |
| 初期化 | `mcp_firebase-mcp-server_firebase_init` |
| セキュリティルール | `mcp_firebase-mcp-server_firebase_get_security_rules` |
| 環境設定 | `mcp_firebase-mcp-server_firebase_update_environment` |

---

## Antigravity MCP ツール統合

### コアツール

| 用途 | ツール |
| ---- | ------ |
| ファイル読み込み | `view_file`, `mcp_filesystem_read_text_file` |
| ファイル書き込み | `write_to_file`, `mcp_filesystem_write_file` |
| ファイル編集 | `replace_file_content`, `multi_replace_file_content` |
| ファイル検索 | `find_by_name`, `grep_search` |
| コマンド実行 | `run_command` |
| タスク管理 | `task_boundary` |
| サブエージェント | `browser_subagent` |
| 画像生成 | `generate_image` |

### Playwright MCP（ブラウザ）

| 操作 | コマンド |
| ---- | -------- |
| URLに移動 | `mcp_playwright_browser_navigate` |
| ページ構造 | `mcp_playwright_browser_snapshot` |
| クリック | `mcp_playwright_browser_click` |
| テキスト入力 | `mcp_playwright_browser_type` |
| フォーム入力 | `mcp_playwright_browser_fill_form` |
| スクリーンショット | `mcp_playwright_browser_take_screenshot` |
| コンソール | `mcp_playwright_browser_console_messages` |
| ネットワーク | `mcp_playwright_browser_network_requests` |
| リサイズ | `mcp_playwright_browser_resize` |
| JS実行 | `mcp_playwright_browser_evaluate` |
| 待機 | `mcp_playwright_browser_wait_for` |

### Context7 MCP（ドキュメント検索）

実装・デバッグ時にライブラリの最新ドキュメントを自動参照:

```
1. mcp_context7_resolve-library-id → ライブラリID取得
2. mcp_context7_query-docs → 具体的な質問でドキュメント取得
```

**使用タイミング**: ultra-brainstorm / ultra-plan / ultra-implement / ultra-debug / ultra-onboard

### Memory MCP（知識永続化）

セッション間でプロジェクト知識を保持:

```
mcp_memory_create_entities   → 新規知識の保存
mcp_memory_search_nodes      → 過去の知識検索
mcp_memory_add_observations  → 既存知識の更新
mcp_memory_create_relations  → 知識間の関係記録
```

**使用タイミング**: ultra-onboard / ultra-retro / ultra-debug

### Draw.io MCP（アーキテクチャ図）

```
mcp_drawio_open_drawio_mermaid → Mermaid記法で図生成
mcp_drawio_open_drawio_xml     → 詳細な図（XML形式）
```

**使用タイミング**: ultra-plan / ultra-onboard

### GitHub MCP

| 操作 | ツール |
| ---- | ------ |
| PR作成 | `mcp_github_create_pull_request` |
| ブランチ作成 | `mcp_github_create_branch` |
| ファイル取得 | `mcp_github_get_file_contents` |
| Issue操作 | `mcp_github_create_issue` / `mcp_github_list_issues` |

---

## レスポンシブテスト手順

```
1. mcp_playwright_browser_navigate → 対象URL
2. mcp_playwright_browser_resize → {width: 1920, height: 1080}  # Desktop
3. mcp_playwright_browser_take_screenshot → desktop.png
4. mcp_playwright_browser_resize → {width: 768, height: 1024}   # Tablet
5. mcp_playwright_browser_take_screenshot → tablet.png
6. mcp_playwright_browser_resize → {width: 375, height: 812}    # Mobile
7. mcp_playwright_browser_take_screenshot → mobile.png
```

---

## Eng Review 認知パターン（15原則）

1. **State Diagnosis** — チームの状態を見極める
2. **Blast Radius** — 最悪のケースの影響範囲
3. **Boring by Default** — 実績のある技術を優先
4. **Incremental over Revolutionary** — Strangler Fig、Big Bangではなく
5. **Systems over Heroes** — 深夜3時の疲れた人間向けに設計
6. **Reversibility Preference** — Feature flags、canary rollout
7. **Failure is Information** — Blameless postmortems
8. **Org = Architecture** — Conway's Law
9. **DX is Quality** — 遅いCI/デプロイ → 品質低下
10. **Essential vs Accidental Complexity** — Brooks
11. **Two-week Smell Test** — 2週間で小機能出せないなら問題あり
12. **Glue Work Awareness** — 見えない調整作業の認識
13. **Make the Change Easy, Then Make the Easy Change** — Beck
14. **Own Your Code in Production** — Dev/Ops の壁なし
15. **Error Budgets** — SLO = ダウンタイム予算

---

## Git Worktree 管理

1. 既存の `.worktrees/` or `worktrees/` を確認
2. なければユーザーに確認して作成
3. `.gitignore` に追加されていることを検証
4. worktree内でテストベースライン確認
5. 完了後は `git worktree remove` でクリーンアップ

---

## スキルパス解決

```
ultrapower/skills/{skill-name}/SKILL.md
```

スキル参照は「ultrapower:{skill-name}」形式。

---

## アーティファクト保存場所

```
docs/ultrapower/
├── specs/                ← ultra-brainstorm（設計仕様書）
├── designs/              ← ultra-brainstorm / ultra-design-system
├── plans/                ← ultra-plan
├── reviews/              ← ultra-review
├── qa-reports/           ← ultra-qa
├── ship-logs/            ← ultra-ship
├── retro-reports/        ← ultra-retro
└── benchmark-reports/    ← ultra-benchmark
```

各スキル実行時に `mkdir -p` で自動作成。

---

## 完了状態レポート形式

```
╔══════════════════════════════════════════╗
║  ULTRAPOWER — [フェーズ名] 完了          ║
╠══════════════════════════════════════════╣
║  ステータス: [PASS/FAIL/PARTIAL]         ║
║  実施項目:   [N] 件                      ║
║  検出問題:   [N] 件（解決: [N]）         ║
║  次のフェーズ: [推奨フェーズ名]           ║
╚══════════════════════════════════════════╝
```
