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
| ultra-second-opinion | 別AIへの二次意見ブリッジ |
| ultra-parallel | 独立タスクの並列実行・browser_subagent委譲・バックグラウンドパイプライン |
| ultrapower-workflow | 常時（自動ルーティング） |

**明確に関連するスキルがあればトリガーすること。判断に迷う場合はユーザーに確認。**

---

## 5つの鉄則

### 1. テストファースト（TDD厳守）
失敗するテストを書いてから本番コードを書く。例外なし。

**TDD適用除外** — 以下はテストファーストを適用しない代わりに、指定の検証で品質を担保:

| 変更種別 | 代替検証 |
|----------|----------|
| CSS / スタイリング | ultra-design-review で視覚検証（スクリーンショット比較） |
| 設定ファイル (.env, firebase.json 等) | 動作確認（Dev Server起動 or コマンド実行） |
| ドキュメント更新 | ultra-docs の一貫性チェック |

**⚠️ TDD適用除外にならないもの:**

- ❌ 「プロトタイプだから」— 本番デプロイするコードは必ずテストが必要
- ❌ 「MVPだから」— MVP こそバグが致命的（初期ユーザーの信頼を失う）
- ❌ 「急いでいるから」— テストなしのデプロイは結果的に遅くなる（デバッグコスト）
- ❌ 「Firebase/Supabase のルールだけだから」— セキュリティルールこそテスト必須

> **教訓（Small World MVP — 2026-03-25）**: 「素早いプロトタイプ」としてテストをスキップした結果、本番環境でFirestoreサブコレクションの権限エラーが発生。エミュレータテストがあれば防げたバグに、3回のデプロイサイクルを費やした。

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

```text
╔══════════════════════════════════════════╗
║  STATUS: [DONE / DONE_WITH_CONCERNS /   ║
║          BLOCKED / NEEDS_CONTEXT]        ║
║  REASON: [1-2行の説明]                   ║
║  EVIDENCE: [検証コマンド出力 or リンク]   ║
╚══════════════════════════════════════════╝
```

**ルール**: DONE 以外のステータスでは必ず REASON を記載すること。

---

## コードブロック言語指定ルール

SKILL.md 内のコードブロックには必ず言語を指定する:

| 用途 | 言語指定 |
|------|---------|
| MCP ツール呼び出し | `yaml` |
| シェルコマンド | `bash` |
| JavaScript/TypeScript | `javascript` / `typescript` |
| 出力例・テーブル | `text` |
| Mermaid 図 | `mermaid` |
| Python | `python` |
| Markdown テンプレート | `markdown` |

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

```yaml
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

```yaml
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

### DB不使用プロジェクト（Noneパス）

上記のシグナルがいずれも検出されず、プロジェクトがDBを使用しない場合:

```yaml
DB BACKEND: None
スキップ: DB関連のステップ（マイグレーション、セキュリティルール等）
対象: 静的サイト、CLIツール、ライブラリ、フロントエンドのみPJ等
```

**アクション**: ultra-onboard / ultra-implement / ultra-ship でDB関連ステップをスキップ。

### Supabase プロジェクト

```yaml
DB操作:      Supabase Dashboard or supabase CLI
認証:        Supabase Auth (@supabase/supabase-js)
ストレージ:  Supabase Storage
マイグレーション: supabase/migrations/
デプロイ:    Vercel / Cloudflare (フロント) + Supabase (バックエンド)
```

#### Supabase マイグレーション自動化（推奨）

手動SQL Editorではなく、マイグレーションファイルで管理:

```bash
# 1. マイグレーションファイル作成
mkdir -p supabase/migrations
# ファイル名: XXX_description.sql (例: 001_init.sql, 002_add_tags.sql)

# 2. Supabase CLI が利用可能な場合
npx supabase migration new init     # マイグレーション作成
npx supabase db push                 # リモートに適用

# 3. CLI が使えない場合の手順
# supabase/migrations/ にSQLファイルを配置
# Supabase Dashboard SQL Editor で手動実行
# ⚠️ 実行後、必ず結果をスクリーンショットで記録
```

#### Supabase キーバリデーション

`.env` のAPI キーは起動時にバリデーションすること:

- JWTフォーマット確認（header.payload.signature の3パート）
- issuer が `supabase` であること
- バッククォート・不正文字の混入チェック
- 不正時はコンソールエラー + UIトースト通知

### Firebase プロジェクト

```yaml
DB操作:      mcp_firebase-mcp-server_* ツール群
認証:        Firebase Auth (mcp_firebase-mcp-server_firebase_init → auth)
ストレージ:  Firebase Storage
マイグレーション: Firestore Security Rules
デプロイ:    mcp_firebase-mcp-server_firebase_init → hosting
```

#### Firebase セットアップ自動化

```bash
# 1. プロジェクト設定
mcp_firebase-mcp-server_firebase_update_environment:
  project_dir: [プロジェクトディレクトリ]
  active_project: [Firebase プロジェクトID]

# 2. サービス初期化
mcp_firebase-mcp-server_firebase_init:
  features:
    firestore: { location_id: "asia-northeast1" }
    auth: { providers: { emailPassword: true } }
    hosting: { public_directory: "dist" }

# 3. セキュリティルール確認
mcp_firebase-mcp-server_firebase_get_security_rules:
  type: "firestore"
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

### デュアルDB 抽象化パターン（推奨）

> **高度な使用法** — 通常は Supabase か Firebase の単体使用で十分。両方のバックエンドを切替可能にする場合のみ参照。

Supabase と Firebase を同一プロジェクトで切替可能にする Provider Pattern:

```javascript
// db.js — エントリーポイント
const DB_PROVIDER = import.meta.env.VITE_DB_PROVIDER || 'supabase';

export async function getDB() {
  if (DB_PROVIDER === 'firebase') {
    const { FirebaseAdapter } = await import('./adapters/firebase-adapter.js');
    return new FirebaseAdapter();
  }
  const { SupabaseAdapter } = await import('./adapters/supabase-adapter.js');
  return new SupabaseAdapter();
}
```

#### 統一 API 契約

```javascript
// 認証
db.auth.getUser()        // → User | null
db.auth.signUp(email, pw)
db.auth.signIn(email, pw)
db.auth.signOut()

// データ操作（Supabase互換チェーンAPI）
db.from('table').select('*').eq('field', value).order('created_at', { ascending: false })
db.from('table').insert(record)
db.from('table').update(record).eq('id', docId)
db.from('table').delete().eq('id', docId)
```

#### 切替方法

```bash
# .env
VITE_DB_PROVIDER=supabase  # デフォルト
VITE_DB_PROVIDER=firebase  # Firebase に切替
```

**実績**: StudyFlow PJで検証済み — 83テスト全PASS、ページコード変更なしで切替可能

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

```text
1. mcp_context7_resolve-library-id → ライブラリID取得
2. mcp_context7_query-docs → 具体的な質問でドキュメント取得
```

**使用タイミング**: ultra-brainstorm / ultra-plan / ultra-implement / ultra-debug / ultra-onboard

### Memory MCP（知識永続化）

セッション間でプロジェクト知識を保持:

```text
mcp_memory_create_entities   → 新規知識の保存
mcp_memory_search_nodes      → 過去の知識検索
mcp_memory_add_observations  → 既存知識の更新
mcp_memory_create_relations  → 知識間の関係記録
```

**使用タイミング**: ultra-onboard / ultra-retro / ultra-debug

### Draw.io MCP（アーキテクチャ図）

```text
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

### Hugging Face（Pro）

> 100万+学習済みモデル、25万+データセット、公開デモ（Spaces）を活用するAIエコシステム

- **アカウント**: `yuuya-miyagaki`（Pro契約）
- **認証**: `~/.cache/huggingface/token` に保存済み（自動読み込み）
- **CLI**: `hf` コマンド（`~/.zshrc` にPATH設定済み）
- **SDK**: Python `huggingface_hub` / JS `@huggingface/inference`

#### やりたいこと → 使い方

| やりたいこと | HF機能 | コード例 |
|-------------|--------|---------|
| テキスト生成・チャット | Inference API | `InferenceClient().chat.completions.create(model="meta-llama/Llama-3.1-8B-Instruct", ...)` |
| 画像生成 | Inference API | `InferenceClient().text_to_image("prompt", model="stabilityai/stable-diffusion-xl-base-1.0")` |
| 翻訳・要約・感情分析 | Inference API | `InferenceClient().translation("テキスト", model="Helsinki-NLP/opus-mt-ja-en")` |
| 音声→テキスト | Inference API | `InferenceClient().automatic_speech_recognition("audio.mp3", model="openai/whisper-large-v3")` |
| 学習済みモデルをローカル実行 | Models Hub | `pipeline("task", model="model-name")` ※transformers |
| 公開データセットを利用 | Datasets Hub | `load_dataset("dataset-name")` ※datasets |
| 他者の公開デモをAPI利用 | Spaces | `Client("space-owner/space-name").predict(...)` ※gradio_client |
| WebアプリにAI機能を組込み | JS SDK | `import { HfInference } from "@huggingface/inference"` |

#### Models Hub — 探索先行で活用

モデル選定時は [huggingface.co/models](https://huggingface.co/models) で検索。タスク・言語・ライセンスでフィルタ可能。
車輪の再発明を避けるため、**実装前に既存モデルの有無を確認**すること。

#### Datasets Hub — サンプルデータ・学習データとして活用

[huggingface.co/datasets](https://huggingface.co/datasets) でデータセット検索。アプリのデモデータやプロトタイプ用データとしても利用可。

#### Spaces — 既存デモの再利用

[huggingface.co/spaces](https://huggingface.co/spaces) に公開されたMLデモは `gradio_client` 経由でAPIとして呼び出せる。
自前実装の前に、**同等機能の公開 Space がないか探索**すること。

---

## レスポンシブテスト手順

```text
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

```text
ultrapower/skills/{skill-name}/SKILL.md
```

スキル参照は「ultrapower:{skill-name}」形式。

---

## アーティファクト保存場所

```text
docs/ultrapower/
├── specs/                ← ultra-brainstorm（設計仕様書）
├── designs/              ← ultra-brainstorm / ultra-design-system
├── plans/                ← ultra-plan
├── reviews/              ← ultra-review（品質レポート）
├── qa-reports/           ← ultra-qa
├── ship-logs/            ← ultra-ship
├── retro-reports/        ← ultra-retro
├── benchmark-reports/    ← ultra-benchmark
└── design-reviews/       ← ultra-design-review（デザイン監査レポート）
```

各スキル実行時に `mkdir -p` で自動作成。

---

## 完了状態レポート形式

> 各スキル固有の完了レポートは各 SKILL.md 内で定義。共通ステータスコードは「完了ステータスプロトコル」セクション（上記）を参照。

```text
╔══════════════════════════════════════════╗
║  ULTRAPOWER — [フェーズ名] 完了          ║
╠══════════════════════════════════════════╣
║  STATUS: [DONE / DONE_WITH_CONCERNS /   ║
║          BLOCKED / NEEDS_CONTEXT]        ║
║  実施項目:   [N] 件                      ║
║  検出問題:   [N] 件（解決: [N]）         ║
║  次のフェーズ: [推奨フェーズ名]           ║
╚══════════════════════════════════════════╝
```

---

## 検証責務マトリクス

各検証項目のオーナーシップを明確化。**Owner** が正式な検証を行い、他スキルでの実行は「前提確認」。

| 検証項目 | Owner（正式検証） | 前提確認のみ |
|----------|------------------|-------------|
| 自動テスト実行 | **ultra-qa** | ultra-implement（開発中）, ultra-ship（最終確認） |
| コードレビュー | **ultra-review** | — |
| セキュリティ監査（OWASP/STRIDE） | **ultra-ship** | ultra-onboard（概観のみ） |
| 依存脆弱性（npm audit等） | **ultra-ship** | ultra-onboard（概観のみ） |
| レスポンシブUI検証 | **ultra-qa** | ultra-design-review（デザイン観点） |
| パフォーマンス計測 | **ultra-benchmark** | ultra-qa（簡易ロード時間のみ） |
| デザイン品質検証 | **ultra-design-review** | — |
| DB検出 | **ultra-onboard** | ultra-implement, ultra-ship（結果を参照） |
