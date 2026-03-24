---
name: ultra-implement
description: "計画承認後に起動。TDDサイクルで計画を実行し、UI検証にPlaywright MCPを使用する。タスク分割と2段階レビュー（仕様準拠→コード品質）で大規模計画にも対応。"
---

# Ultra Implement — 統合実装

承認された実装計画をTDDサイクルで実行する。

> **学習パターン参照**: 同ディレクトリの `learned-patterns.md` が存在する場合、スキル実行時に参照し、過去の知見を活用する。

## 前提ルール（AGENTS.md「5つの鉄則」参照）

```text
失敗するテストなしに本番コードを書かない
```

コードを先に書いた？ **削除。最初からやり直す。**
- 「参考にする」のもダメ
- 「アダプトする」のもダメ
- 削除は削除

## Step 1: Context7 でドキュメント準備

実装で使用するライブラリの最新ドキュメントを事前取得:

```yaml
mcp_context7_resolve-library-id: [使用ライブラリ名]
mcp_context7_query-docs: [実装に必要な具体的なAPI/パターン]
```

**例**: Next.js App Router, Supabase Auth, Firebase SDK 等

## Step 2: 環境準備

```bash
# Git Worktree で隔離環境を作成（オプション・推奨）
git worktree add ../project-feature feature/feature-name 2>/dev/null || true

# 依存関係インストール
npm install  # / pip install -r requirements.txt / cargo build

# テストベースライン確認
npm test  # / pytest / cargo test / go test ./...
```

### DB バックエンド確認

AGENTS.md の自動検出ルールに従ってDB種別を特定（詳細は AGENTS.md「DB バックエンド選択」セクション参照）。

## Step 3: 実行モード選択

計画のタスク構成に応じて実行モードを選択:

### モード A: 直接TDDサイクル（デフォルト）

タスクが少数（1-5個）または密結合の場合。→ Step 4 へ直接進む。

### モード B: タスク分割＋逐次TDD実行（大規模計画向け）

タスクが多数（5つ以上）かつ概ね独立している場合に使用:

```yaml
選択基準:
  ✅ 実装計画がある → YES
  ✅ タスクが概ね独立 → YES
  ✅ タスクが5つ以上 → 推奨
```

#### タスク分割実行フロー

```text
計画読み込み → タスク抽出 → 依存順にソート

  ┌─────── Per Task ───────┐
  │ 1. TDDサイクル実行       │
  │ 2. 仕様準拠レビュー      │
  │ 3. コード品質レビュー    │
  │ 4. コミット＋タスク完了   │
  └──────────────────────┘

全タスク完了 → 統合テスト → ultra-review へ
```

#### 1. タスクの逐次TDD実行

各タスクを1つずつ Step 4 の TDDサイクルで実装。**1タスク完了後に次タスクへ進む**（並列実行禁止）。

#### 2. 仕様準拠レビュー（各タスク完了後）

タスクの仕様書と実際のコード変更を照合:
- 計画の全要件が実装されているか？
- 計画にない余分な機能が追加されていないか？（YAGNI）
- 不合格 → 修正 → 再レビュー

#### 3. コード品質レビュー（仕様準拠後）

仕様準拠確認後にコード品質をチェック:
- テストカバレッジ
- 命名規約
- エラーハンドリング
- パフォーマンス
- 不合格 → 修正 → 再レビュー

**重要:** 仕様準拠✅ → コード品質の順。逆順は禁止。

#### 4. ブロック時の対応

| 状況 | 対応 |
|------|------|
| タスクが大きすぎる | さらに小タスクに分割 |
| 依存タスクが未完了 | 依存タスクを先に実装 |
| 計画に問題がある | ユーザーにエスカレーション |

**絶対禁止:** 問題を無視して同じことを再試行（3回失敗でエスカレーション）

## Step 4: TDDサイクルでタスク実行

計画の各タスクに対して、厳密にRED → GREEN → REFACTORを実行。

### モジュールパターンの注意（Vanilla JS + Jest）

Vanilla JavaScript プロジェクトで Jest テストを書く場合、`module.exports` はブラウザで `ReferenceError` を引き起こす。**UMD互換パターン**を使用すること:

```javascript
// ✅ UMD互換 — ブラウザとNode.js両方で動作
// ファイル末尾に追加:
if (typeof module !== 'undefined' && module.exports) {
  module.exports = { MyClass };
}

// ❌ ブラウザでエラーになる
module.exports = { MyClass };
```

**適用場面**: フレームワーク不使用のHTML+JSプロジェクト。React/Next.js/Vite等のバンドラー使用時は不要。

### RED — 失敗するテストを書く

```typescript
// 1つの振る舞いをテスト。明確な名前。
test('正しい認証情報でログインできる', async () => {
  const result = await login('user@test.com', 'password');
  expect(result.success).toBe(true);
  expect(result.token).toBeDefined();
});
```

**要件:**
- 1テスト = 1つの振る舞い
- テスト名が振る舞いを説明
- 実コード使用（モックは最小限）

### テスト失敗を確認（必須・省略不可）

```bash
npm test path/to/test.test.ts
```

確認事項:
- テストが**失敗**する（エラーではなく失敗）
- 失敗理由が期待通り（機能未実装のため）
- テストが既に通る場合 → テストを修正

### GREEN — 最小限のコード

テストを通す最小限の実装だけを書く。

**良い例:** テストを通すのに必要な最低限のコード
**悪い例:** 将来のために追加機能も実装（YAGNI違反）

### テスト通過を確認（必須）

```bash
npm test path/to/test.test.ts
# + 全テストスイート
npm test
```

確認事項:
- 該当テスト通過
- 他のテストも全通過
- 警告・エラーなし

### REFACTOR — クリーンアップ

テストが全てグリーンの状態でのみ:
- 重複除去
- 命名改善
- ヘルパー抽出

**振る舞いは変えない。テストをグリーンに保つ。**

### コミット

```bash
git add -A
git commit -m "feat: <タスク名> — RED→GREEN→REFACTOR完了"
```

## Step 5: UI検証（Webアプリの場合）

Playwright MCP でUIの動作を確認：

```yaml
1. mcp_playwright_browser_navigate → http://localhost:3000
2. mcp_playwright_browser_snapshot → インタラクティブ要素を確認
3. mcp_playwright_browser_click → ボタンをクリック
4. mcp_playwright_browser_snapshot → 変化を確認
5. mcp_playwright_browser_take_screenshot → 証拠スクリーンショット
```

### フォームテスト

```text
1. mcp_playwright_browser_navigate → フォームページ
2. mcp_playwright_browser_snapshot → フォーム要素の確認
3. mcp_playwright_browser_fill_form → テストデータ入力
4. mcp_playwright_browser_click → 送信ボタン
5. mcp_playwright_browser_snapshot → 結果確認
6. mcp_playwright_browser_console_messages → エラーチェック
```

## Step 6: 進捗確認と次タスク

各タスク完了後:
1. タスクチェックリストを更新
2. コミット
3. 次のタスクのTDDサイクルへ

## Step 7: 知識の永続化（Memory MCP）

実装中に得た重要な知識をMemoryに保存:

```yaml
mcp_memory_create_entities:
  - name: "[プロジェクト名]-impl-[機能名]"
    entityType: "implementation_knowledge"
    observations:
      - "パターン: [使用したデザインパターン]"
      - "ハマりポイント: [解決に時間がかかった点]"
      - "DB: [Supabase/Firebase] — [具体的な設定/注意点]"
```

## Step 8: 全タスク完了 → ultra-review へ

全タスクのTDDサイクルが完了したら:

```bash
# 最終テスト確認
npm test  # / pytest / cargo test / go test ./...

# テストカバレッジ確認（可能な場合）
npm run test:coverage 2>/dev/null || true
```

全テスト通過 → **ultra-review** スキルに遷移。

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-IMPLEMENT 完了                    ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  タスク数:      [N] 件                   ║
║  TDDサイクル:   [N] 回                   ║
║  テスト追加:    [N] 件                   ║
║  テスト全通過:  [Y/N]                    ║
║  コミット数:    [N]                      ║
║  Memory保存:    [N] 件                   ║
║  次のフェーズ:  ultra-review             ║
╚══════════════════════════════════════════╝
```

## デュアルDB 実装パターン

プロジェクトが Supabase + Firebase のデュアルDB構成の場合:

### 1. Adapter 実装順序

```text
1. Provider Interface 定義 (db.js)
2. SupabaseAdapter テスト → 実装
3. FirebaseAdapter テスト → 実装
4. ページコードの import 切替 (supabase → getDB())
5. テストのモック切替 (supabase.js → db.js)
```

### 2. テストモック切替パターン

```javascript
// jest.mock 内で参照するオブジェクトはファクトリ外で定義
const mockDB = {
  auth: { getUser: jest.fn() },
  from: jest.fn(),
};

jest.mock('../src/db.js', () => ({
  getDB: jest.fn(() => Promise.resolve(mockDB)),
  getAuth: jest.fn(() => Promise.resolve(mockDB.auth)),
}));
```

### 3. 環境変数切替

```bash
VITE_DB_PROVIDER=supabase  # or firebase
```

### 4. 新PJ クイックスタート

新プロジェクトにデュアルDB構成を導入する際のチェックリスト:

```text
[ ] npm install @supabase/supabase-js firebase
[ ] src/db.js — Provider Interface 作成
[ ] src/adapters/supabase-adapter.js — CRUD + Auth
[ ] src/adapters/firebase-adapter.js — CRUD + Auth
[ ] .env — VITE_DB_PROVIDER + 各DB設定値
[ ] firebase.json — auth プロバイダー設定
[ ] firebase deploy --only auth
[ ] テスト — db.js モック基盤
```

## レッドフラグ

**絶対禁止:**

- テストなしに本番コードを書く（TDD厳守）
- レビューをスキップ（仕様準拠もコード品質も必須）
- 未修正の問題を残したまま次タスクへ
- 仕様準拠前にコード品質レビューを実行（順序厳守）
- 同じ問題に3回以上挑戦（エスカレーション必須）
