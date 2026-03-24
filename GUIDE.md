# Ultrapower スキル使い方ガイド

> 各スキルの詳細な使い方、引数、コマンド例、実践Tips

---

## 目次

1. [ultrapower-workflow](#ultrapower-workflow) — 自動ルーティング
2. [ultra-onboard](#ultra-onboard) — コードベース理解
3. [ultra-brainstorm](#ultra-brainstorm) — アイデア整理
4. [ultra-design-system](#ultra-design-system) — デザインシステム
5. [ultra-plan](#ultra-plan) — 実装計画
6. [ultra-implement](#ultra-implement) — TDD実装
7. [ultra-review](#ultra-review) — コードレビュー
8. [ultra-qa](#ultra-qa) — テスト
9. [ultra-benchmark](#ultra-benchmark) — パフォーマンス
10. [ultra-design-review](#ultra-design-review) — デザイン監査
11. [ultra-ship](#ultra-ship) — デプロイ
12. [ultra-debug](#ultra-debug) — デバッグ
13. [ultra-retro](#ultra-retro) — 振り返り
14. [ultra-docs](#ultra-docs) — ドキュメント
15. [ultra-second-opinion](#ultra-second-opinion) — 二次意見

---

## ultrapower-workflow

**自動フェーズルーティング + 安全ガードレール**

フレームワークの「司令塔」。ユーザーの意図を解析して適切なスキルに自動ルーティングする。

### 発動条件

- 自然言語での開発要望
- 「新しいアプリを作りたい」「バグがある」「デプロイして」

### ルーティング例

| ユーザー入力 | ルーティング先 |
|-------------|--------------|
| 「To-Doアプリを作って」 | brainstorm → plan → implement |
| 「このバグを直して」 | debug |
| 「本番にデプロイ」 | ship |
| 「リファクタリングしたい」 | plan → implement → review |

---

## ultra-onboard

**既存コードベースを理解する**

### 起動

```
「このリポジトリを理解したい」
「コードベースの全体像を教えて」
```

### 出力

- アーキテクチャ図（Draw.io）
- DB構造分析
- 技術スタック特定
- Memory MCPに知識永続化

### Tips

- 大規模プロジェクトでは先に `ls` や `find` でディレクトリ構造を把握してからonboardを実行
- 既にMemoryに知識がある場合は差分更新のみ実行

---

## ultra-brainstorm

**新プロジェクトのアイデア整理 + 初期設定**

### 起動

```
「新しいアプリのアイデアを考えたい」
「〇〇なアプリを作りたい」
```

### 出力

- git init + .gitignore
- `docs/ultrapower/specs/spec.md`（仕様書）
- アイデア整理ドキュメント

### Tips

- 技術スタックの指定がない場合、最適なものを提案
- DB選択（Supabase/Firebase/None）も自動判定

---

## ultra-design-system

**デザインシステム構築**

### 起動

```
「デザインを決めよう」
「UIのスタイルを作りたい」
```

### 出力

- `DESIGN.md`（デザイントークン定義）
- CSSファイル（style.css / index.css）
- プレビューHTML

### Tips

- 競合サイトのリサーチ結果を基に提案
- SAFE/RISKの2案を提示し、ユーザーが選択

---

## ultra-plan

**TDD実装計画**

### 起動

```
「実装計画を立てて」
「この機能をどう実装する？」
```

### レビューモード

| モード | 条件 | レビュー |
|-------|------|---------|
| Full | 大規模変更 | CEO → Eng → Design 3段階 |
| Light | 小規模変更 | 単一統合レビュー |

### 出力

- 実装計画書（タスク分割）
- アーキテクチャ図（Draw.io）
- Context7によるAPI ドキュメント確認

---

## ultra-implement

**TDDサイクルで計画を実行**

### 起動

計画承認後に自動遷移、または:

```
「実装を開始して」
「このタスクを実装して」
```

### TDD サイクル

```
RED   → 失敗するテストを書く
GREEN → 最小限のコードでテストを通す
REFACTOR → クリーンアップ（テストはグリーンのまま）
```

### 実行モード

| モード | 条件 |
|-------|------|
| A: 直接TDD | タスク1-5個、密結合 |
| B: タスク分割 | タスク5+個、独立性あり |

### Tips

- `jest.mock()` でSupabase/Firebase をモック
- Vite + ESM の場合は `babel-jest` が必要
- 大量コードの一括コミットは避ける（component別に分割）

---

## ultra-review

**検証駆動コードレビュー**

### 起動

```
「レビューして」
「コードを確認して」
```

### プロセス

1. **仕様準拠レビュー** — 計画通りか？
2. **コード品質レビュー** — 命名、エラー処理、パフォーマンス
3. **Fix-First** — 問題があれば修正 → 再レビュー

### 鉄則

> 「検証なき完了宣言禁止」— テスト実行せずに「完了」と言ってはならない

---

## ultra-qa

**テスト実行 + ブラウザQA**

### 起動

```
「テストして」
「QAをお願い」
```

### 対応テスト

| テスト種別 | ツール |
|-----------|-------|
| ユニットテスト | Jest / pytest / cargo test |
| ブラウザテスト | Playwright MCP |
| レスポンシブ | Playwright + viewport変更 |

### Tips

- Dev Server が起動していなければ自動起動
- スクリーンショットを証拠として保存

---

## ultra-benchmark

**パフォーマンス計測**

### 起動

```
「パフォーマンスを計測して」
「ベンチマークを取って」
```

### 引数

```
/benchmark              デフォルト計測
/benchmark --baseline   ベースライン保存
/benchmark --compare    前回との比較
```

### 計測項目

- Core Web Vitals (TTFB, FCP, LCP, CLS, INP)
- バンドルサイズ (gzip含む)
- ビルド時間
- Navigation Timing API

---

## ultra-design-review

**デザイン監査**

### 起動

```
「デザインを確認して」
「UIをレビューして」
```

### 7軸診断

1. カラーパレット
2. タイポグラフィ
3. スペーシング
4. コンポーネント一貫性
5. レスポンシブ
6. アクセシビリティ
7. AIスロップ検出

---

## ultra-ship

**セキュリティ監査 + デプロイ**

### 起動

```
「デプロイして」
「本番にリリースしたい」
```

### プロセス

1. `npm audit` — 依存パッケージ脆弱性チェック
2. OWASP/STRIDE — セキュリティ監査
3. ビルド確認 — `npm run build`
4. デプロイ実行 — Vercel / Firebase Hosting

### Tips

- Supabase プロジェクト → Vercel にデプロイ（環境変数設定忘れに注意）
- Firebase プロジェクト → `firebase deploy`

---

## ultra-debug

**4フェーズ体系的デバッグ**

### 起動

バグ発見時に自動、または:

```
「このバグを直して」
「エラーが出る」
```

### 4フェーズ

1. **再現** — 問題を安定的に再現
2. **仮説** — 原因の仮説を立てる（最大3つ）
3. **検証** — 仮説を1つずつテスト
4. **修正** — テスト付きで修正

### 鉄則

> 3回失敗したらエスカレーション（同じ問題に4回目の挑戦禁止）

---

## ultra-retro

**Git分析レトロスペクティブ**

### 起動

```
「振り返りしよう」
「何がうまくいった？」
```

### 引数

| 引数 | 期間 |
|------|------|
| `/retro` | 7日間（デフォルト） |
| `/retro 24h` | 過去24時間 |
| `/retro 14d` | 過去14日間 |
| `/retro compare` | 前回との比較 |
| `/retro global` | 複数リポジトリ横断 |

### 出力

- KEEP / IMPROVE / STOP の3カテゴリ
- メトリクスサマリー（コミット数、LOC、テスト比率等）
- ストリーク追跡
- JSON スナップショット保存
- Memory MCP に永続化

---

## ultra-docs

**ドキュメント自動更新**

### 起動

```
「ドキュメントを更新して」
「READMEを最新にして」
```

### プロセス

1. Git diff 分析
2. 影響を受けるドキュメントを特定
3. 自動更新
4. 一貫性チェック

---

## ultra-second-opinion

**二次意見ブリッジ**

### 起動

```
「別の意見が欲しい」
「Claude に聞いてみたい」
```

### 出力

- コンテキスト自動収集（関連ファイル、エラー、テスト結果）
- Claude Code向けコピペ用プロンプト生成
- 回答の統合提案

---

## 安全ガードレール

全スキルに共通する安全機構:

| ガードレール | 内容 |
|------------|------|
| **See Something Say Something** | 問題を発見したら即報告 |
| **3回ルール** | 同じ問題に3回失敗 → エスカレーション |
| **TDD厳守** | テストなしに本番コードを書かない |
| **検証なき完了禁止** | テスト実行せずに完了宣言しない |
| **Boil the Lake** | 部分的な実装で妥協しない |
