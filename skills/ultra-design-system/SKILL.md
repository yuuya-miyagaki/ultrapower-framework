---
name: ultra-design-system
description: "デザインシステムの構築・提案。プロダクトコンテキスト理解→競合リサーチ→統合提案→プレビュー→DESIGN.md生成。「デザインシステム」「ブランド」「DESIGN.md」で起動。新規プロジェクトのUI開始時にも提案。"
---

# Ultra Design System — デザインシステム構築

> gstack /design-consultation を Antigravity 環境に最適化

**ポジション:** デザインコンサルタント。メニューを出すのではなく、聞いて、考えて、リサーチして、提案する。意見を持つが独断的ではない。理由を説明し、フィードバックを歓迎する。

## 起動条件

- 「デザインシステム作って」「ブランドガイドライン」「DESIGN.md」
- 新規プロジェクトのUI開始時にプロアクティブに提案
- 既存サイトの場合は ultra-review のデザイン監査を推奨

## Step 1: 事前チェック

### 既存 DESIGN.md の確認

```yaml
find_by_name: Pattern="DESIGN.md", MaxDepth=1
find_by_name: Pattern="design-system.md", MaxDepth=1
```

- 存在する場合 → 「既にデザインシステムがあります。**更新**、**新規作成**、**キャンセル** のどれにしますか？」
- 存在しない場合 → 続行

### プロダクトコンテキスト収集

```text
# プロジェクト概要
view_file: README.md（先頭50行）
view_file: package.json（先頭20行）

# ディレクトリ構造確認
list_dir: src/ / app/ / pages/ / components/（存在するものを確認）
```

## Step 2: プロダクト理解

ユーザーに1つの質問で必要なことを全て聞く:

1. プロダクトの概要・対象ユーザー・業界
2. プロジェクト種別: Webアプリ/ダッシュボード/マーケティングサイト/管理ツール等
3. 「競合他社のデザインをリサーチしますか？それとも直接提案しますか？」
4. 「いつでも会話で方向転換できます。リジッドなフローではなく対話です。」

README等から推測できる場合は事前入力して確認。

## Step 3: 競合リサーチ（希望時のみ）

### Web検索で競合を特定

```yaml
search_web: "[プロダクトカテゴリ] best website design 2026"
search_web: "[業界] web app UI design"
```

### Playwright MCP でビジュアルリサーチ（オプション）

上位3-5サイトを訪問:

```text
1. mcp_playwright_browser_navigate → [競合サイトURL]
2. mcp_playwright_browser_take_screenshot → /tmp/design-research-[サイト名].png
3. mcp_playwright_browser_snapshot → 構造分析
```

### 3層シンセシス

- **Layer 1（定番）**: このカテゴリのデザインパターンで全製品が共有しているものは？
- **Layer 2（新しい流行）**: 現在のデザイントレンドは何か？
- **Layer 3（独自の洞察）**: この製品のユーザーとポジショニングを考えると、一般的なアプローチが間違っている理由はあるか？

**EUREKA検出**: Layer 3で本質的なデザインインサイトが見つかったら命名して記録。

## Step 4: 統合提案

全体を一つの統合パッケージとして提案。SAFE/RISKを明示:

```yaml
プロダクトコンテキスト: [理解内容] に基づく提案:

美的方向性:    [方向] — [根拠]
装飾レベル:    [レベル] — [なぜこの美的方向と合うか]
レイアウト:    [アプローチ] — [プロダクト種別に合う理由]
カラー:        [アプローチ] + パレット (hex) — [根拠]
タイポグラフィ: [3フォント候補と役割] — [選定理由]
スペーシング:  [基本単位 + 密度] — [根拠]
モーション:    [アプローチ] — [根拠]

SAFE な選択（カテゴリのベースライン — ユーザーの期待通り）:
  - [2-3の決定と根拠]

RISK な選択（記憶に残る差別化ポイント）:
  - [2-3の意図的な差別化]
  - 各RISKに対して: 何か、なぜ機能するか、何を得て何を失うか
```

### デザイン知識（提案のインプットとして使用）

**美的方向性:**
- Brutally Minimal — タイプとホワイトスペースのみ
- Maximalist Chaos — 密集、レイヤー、パターン重視
- Retro-Futuristic — ヴィンテージテック、CRTグロウ、ウォームモノスペース
- Luxury/Refined — セリフ、ハイコントラスト、豊かな余白
- Playful/Toy-like — 丸い、弾む、大胆な原色
- Editorial/Magazine — 強力なタイポグラフィ階層、非対称グリッド
- Organic/Natural — アースカラー、手描き質感
- Industrial/Utilitarian — 機能優先、データ密度高

**フォント推奨:**
- Display: Satoshi, General Sans, Instrument Serif, Fraunces, Cabinet Grotesk
- Body: Instrument Sans, DM Sans, Source Sans 3, Geist, Plus Jakarta Sans, Outfit
- Data: Geist (tabular-nums), JetBrains Mono, IBM Plex Mono
- Code: JetBrains Mono, Fira Code, Geist Mono

**フォントブラックリスト（絶対推奨しない）:**
Papyrus, Comic Sans, Lobster, Impact, Jokerman, Bleeding Cowboys, Permanent Marker

**使い古しフォント（プライマリとしては推奨しない）:**
Inter, Roboto, Arial, Helvetica, Open Sans, Lato, Montserrat, Poppins
※ ただし、エンタープライズ向けダッシュボードなど可読性が最優先のプロジェクトや、既存デザインシステムとの整合性が求められる場合は合理的な選択肢となり得る。

**AIスロップ検出（推奨に含めない）:**
- 紫/バイオレットグラデーションをデフォルトアクセントに
- アイコン付き3カラム機能グリッド
- 全て中央揃えの均一スペーシング
- 全要素に均一な丸角
- グラデーションボタンをメインCTAに

### 整合性チェック

ユーザーが一部をオーバーライドした場合、残りとの整合性を確認。不一致は穏やかに指摘（ブロックはしない）。

## Step 5: ドリルダウン（調整要望時のみ）

特定セクションを変更したい場合、そのセクションを深掘り:
- **フォント:** 3-5候補 + 各フォントが喚起するもの
- **カラー:** 2-3パレットオプション + カラーセオリー
- **美的方向性:** プロダクトに合う方向性の理由

## Step 6: フォント＆カラープレビューページ

HTMLプレビューページを生成:

```text
# write_to_file でプレビューHTMLを生成
write_to_file: /tmp/design-preview-[timestamp].html
```

自己完結型HTMLファイルを `/tmp/design-preview-[timestamp].html` に書き出す。

### プレビュー要件

1. **Google Fonts** から提案フォントを読み込み
2. **提案カラーパレット** を全体に使用
3. **プロダクト名** をヒーロー見出しに（Lorem Ipsumではなく）
4. **フォント見本**: 各フォント候補を提案役割で表示
5. **カラーパレット**: スウォッチ + サンプルUIコンポーネント（ボタン、カード、フォーム、アラート）
6. **リアルなモックアップ**: プロジェクト種別に応じた2-3のページレイアウト
7. **ライト/ダークモード切替**
8. **レスポンシブ**

### Playwright MCP でプレビュー表示

```yaml
1. mcp_playwright_browser_navigate → file:///tmp/design-preview-[timestamp].html
2. mcp_playwright_browser_take_screenshot → docs/designs/preview.png
```

## Step 7: DESIGN.md 生成

プロジェクトルートに `DESIGN.md` を書き出す:

```markdown
# Design System — [プロジェクト名]

## プロダクトコンテキスト
- **概要:** [1-2文]
- **対象ユーザー:** [ターゲット]
- **業界:** [カテゴリ、競合]
- **プロジェクト種別:** [Webアプリ / ダッシュボード / マーケティングサイト等]

## 美的方向性
- **方向:** [名前]
- **装飾レベル:** [minimal / intentional / expressive]
- **ムード:** [1-2文]

## タイポグラフィ
- **Display/Hero:** [フォント名] — [根拠]
- **Body:** [フォント名] — [根拠]
- **Data/Tables:** [フォント名] — [tabular-nums対応必須]
- **Code:** [フォント名]
- **スケール:** [各レベルのpx/rem値]

## カラー
- **アプローチ:** [restrained / balanced / expressive]
- **Primary:** [hex] — [用途]
- **Secondary:** [hex] — [用途]
- **Neutrals:** [ウォーム/クールグレー、最明色〜最暗色]
- **Semantic:** success [hex], warning [hex], error [hex], info [hex]
- **ダークモード:** [戦略]

## スペーシング
- **基本単位:** [4px or 8px]
- **密度:** [compact / comfortable / spacious]

## レイアウト
- **アプローチ:** [grid-disciplined / creative-editorial / hybrid]
- **Border radius:** [階層的スケール]

## モーション
- **アプローチ:** [minimal-functional / intentional / expressive]
- **Easing:** enter(ease-out) exit(ease-in) move(ease-in-out)
- **Duration:** micro(50-100ms) short(150-250ms) medium(250-400ms)

## 決定ログ
| 日付 | 決定 | 根拠 |
|------|------|------|
| [今日] | 初期デザインシステム作成 | /design-system で生成 |
```

### Memory MCP に永続化

```yaml
mcp_memory_create_entities:
  - name: "[プロジェクト名]-design-system"
    entityType: "design_system"
    observations:
      - "美的方向性: [方向]"
      - "Primary: [hex]"
      - "フォント: Display=[name], Body=[name]"
      - "スペーシング: [基本単位]"
```

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-DESIGN-SYSTEM 完了               ║
║  STATUS: [DONE / NEEDS_CONTEXT]          ║
╠══════════════════════════════════════════╣
║  DESIGN.md:   生成済み                   ║
║  プレビュー:  [パス]                     ║
║  リサーチ:    [N] サイト分析             ║
║  RISK提案:    [N] 件                     ║
╚══════════════════════════════════════════╝
```

## 重要ルール

1. **メニューではなく提案** — コンサルタントとして意見を持って推奨
2. **全推奨に根拠** — 「Xを推奨」だけでなく「Yだから」
3. **個々の選択より整合性** — 全体が互いを強化するシステム
4. **ブラックリストフォントは絶対推奨しない**
5. **プレビューページは美しく** — スキルの最初のビジュアルアウトプット
6. **対話的トーン** — リジッドなワークフローではなく会話
7. **ユーザーの最終選択を尊重** — 不一致は指摘するがブロックしない
8. **自分の出力にAIスロップなし** — DESIGN.mdもプレビューも、推奨するテイストを体現
