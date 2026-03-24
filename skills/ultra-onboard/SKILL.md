---
name: ultra-onboard
description: "新しいプロジェクトやコードベースに最初に触れる時に起動。体系的にコードを理解し、設計意図・依存関係・アーキテクチャをマッピングする。「このコード読んで」「プロジェクト理解して」で起動。"
---

# Ultra Onboard — コードベース理解 + アーキテクチャマッピング

## 起動条件

- 新しいプロジェクトに初めて触れる時
- 「このコード読んで」「全体を把握して」
- 「アーキテクチャを教えて」

## Step 1: プロジェクト概観

### 1.1 構造スキャン

```
# ドキュメントファイル一覧
find_by_name: Pattern="*.md", MaxDepth=2

# パッケージ定義ファイル確認
view_file: package.json / Cargo.toml / requirements.txt / go.mod（存在するものを読む）

# コード規模の把握（ファイル数 → 必要に応じ行数）
run_command: find . -name "*.ts" -o -name "*.tsx" -o -name "*.py" -o -name "*.go" -o -name "*.rs" 2>/dev/null | wc -l
# 大規模プロジェクトの場合、cloc があれば使用:
run_command: cloc . --quiet 2>/dev/null || echo "cloc未インストール。上記ファイル数を参照"
```

### 1.2 ドキュメント確認

- README.md → プロジェクト目的
- AGENTS.md / GEMINI.md / CLAUDE.md → AI設定
- docs/ → 設計ドキュメント
- CHANGELOG.md → 変更履歴

### 1.3 Memory MCP チェック

```
mcp_memory_search_nodes: "[プロジェクト名]"
```

過去にオンボードした記録があれば差分のみ更新。

## Step 2: アーキテクチャ理解

### 2.1 エントリポイント特定

```
grep_search: "main\|app\|index\|server\|handler"
```

### 2.2 レイヤー構造マッピング

```
LAYER MAP:
  UI/Frontend:     [ファイル/ディレクトリ]
  API/Routes:      [ファイル/ディレクトリ]
  Business Logic:  [ファイル/ディレクトリ]
  Data Access:     [ファイル/ディレクトリ]
  Infrastructure:  [ファイル/ディレクトリ]
  Tests:           [ファイル/ディレクトリ]
```

### 2.3 アーキテクチャ図生成（Draw.io MCP）

```
mcp_drawio_open_drawio_mermaid:
  content: |
    graph TD
      subgraph Frontend
        UI[UI Components]
        Pages[Pages/Routes]
      end
      subgraph Backend
        API[API Routes]
        Logic[Business Logic]
        DB[Database]
      end
      Pages --> API
      API --> Logic
      Logic --> DB
```

## Step 3: 技術スタック特定

### 3.1 DB バックエンド検出

AGENTS.md の自動検出ルールに従い、DB種別を特定（詳細は AGENTS.md「DB バックエンド選択」セクション参照）:

```text
DB BACKEND:
  Type: [Supabase / Firebase / PostgreSQL / MySQL / SQLite / MongoDB / None]
```

**デュアルDB構成の検出:**

```bash
# db.js / adapters/ ディレクトリの存在確認
grep_search: "getDB\|DB_PROVIDER\|adapter"
find_by_name: Pattern="*-adapter.js", SearchDirectory="src/adapters"
```

以下のパターンが見つかれば **デュアルDB構成**:

- `src/db.js` — Provider ファクトリ
- `src/adapters/` — Adapter 群
- `.env` に `VITE_DB_PROVIDER` 環境変数

```text
DB BACKEND:
  Type: Dual (Supabase + Firebase)
  Factory: src/db.js
  Adapters: src/adapters/supabase-adapter.js, src/adapters/firebase-adapter.js
  Active: VITE_DB_PROVIDER=[current value]
```

検出結果を記録し、以降のスキル（ultra-implement, ultra-ship等）で活用。

### 3.2 フレームワーク検出

```
FRAMEWORK:
  Frontend: [Next.js / React / Vue / Svelte / Vanilla]
  Backend:  [Express / Hono / FastAPI / Gin / None]
  DB ORM:   [Prisma / Drizzle / Supabase Client / Firebase SDK]
  Auth:     [Supabase Auth / Firebase Auth / NextAuth / Custom]
  Deploy:   [Vercel / Firebase Hosting / Cloudflare / AWS]
```

### 3.3 Context7 ドキュメント取得

検出したフレームワークの最新ドキュメントを取得:

```
mcp_context7_resolve-library-id: [検出されたライブラリ]
mcp_context7_query-docs: [プロジェクトで使用中のパターン]
```

## Step 4: 依存関係＆セキュリティ概観

```bash
# 各言語を独立実行（混合プロジェクト対応）
run_command: npm audit 2>/dev/null; true
run_command: pip-audit 2>/dev/null; true
run_command: cargo audit 2>/dev/null; true
```

> **注:** 各監査コマンドは独立実行。未インストールのツールはスキップされる。

### 脆弱性サマリー

```
VULNERABILITIES:
  Critical: [N]
  High:     [N]
  Medium:   [N]
  Low:      [N]
```

## Step 5: 知識の永続化（Memory MCP）

```
mcp_memory_create_entities:
  - name: "[プロジェクト名]"
    entityType: "project"
    observations:
      - "スタック: [Frontend] + [Backend] + [DB]"
      - "認証: [Auth方式]"
      - "デプロイ: [デプロイ先]"
      - "エントリポイント: [ファイル]"
      - "アーキテクチャ: [レイヤー構造]"
      - "テスト: [テストフレームワーク]"
      - "DB: [種類] via [ORM/SDK]"

mcp_memory_create_relations:
  - from: "[プロジェクト名]"
    to: "[DB種類]"
    relationType: "uses_database"
  - from: "[プロジェクト名]"
    to: "[フレームワーク]"
    relationType: "built_with"
```

## 完了レポート

```
╔══════════════════════════════════════════╗
║  ULTRA-ONBOARD 完了                      ║
║  STATUS: [DONE / NEEDS_CONTEXT]          ║
╠══════════════════════════════════════════╣
║  プロジェクト: [名前]                     ║
║  スタック:     [Frontend+Backend+DB]     ║
║  総行数:       [N] 行                    ║
║  脆弱性:       [N] 件                    ║
║  Memory保存:   [N] エンティティ           ║
║  次の推奨:     ultra-brainstorm/implement ║
╚══════════════════════════════════════════╝
```
