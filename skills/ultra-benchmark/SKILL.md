---
name: ultra-benchmark
description: "パフォーマンス計測・回帰検出。ブラウザツール（Playwright MCP / browser_subagent）でCore Web Vitals、リソース分析、バンドルサイズを計測。「パフォーマンス」「ベンチマーク」「ページ速度」「バンドルサイズ」で起動。"
---

# Ultra Benchmark — パフォーマンス回帰検出

> gstack /benchmark を Antigravity + ブラウザツール（Playwright MCP / browser_subagent）に最適化

## 起動条件

- 「パフォーマンス測って」「ベンチマーク」「ページ速度」
- 「バンドルサイズ」「Web Vitals」「ロード時間」
- PR作成前のパフォーマンスチェック

> **学習パターン参照**: 同ディレクトリの `learned-patterns.md` が存在する場合、スキル実行時に参照し、過去の知見を活用する。

## 引数

- `/benchmark <url>` — フルパフォーマンス監査（ベースライン比較あり）
- `/benchmark <url> --baseline` — ベースライン取得（変更前に実行）
- `/benchmark <url> --quick` — シングルパス計測（ベースライン不要）
- `/benchmark --trend` — 過去のベンチマーク履歴からトレンド表示

## Step 1: セットアップ

```bash
run_command: mkdir -p docs/benchmark-reports/baselines
```

## Step 2: ページパフォーマンスデータ収集

> **ツール選択**: Playwright MCP が利用可能なら使用（細粒度な JS 実行が可能）。未搭載の場合は `browser_subagent` で代替。

対象URLにブラウザツールでアクセスし、パフォーマンスデータを取得:

#### Playwright MCP 使用時

```text
1. mcp_playwright_browser_navigate → <対象URL>
2. mcp_playwright_browser_wait_for → time: 5（ページロード完了待ち）
```

#### browser_subagent 使用時（フォールバック）

```yaml
browser_subagent:
  Task: |
    1. <対象URL> にアクセス
    2. ページロード完了を待つ
    3. DevTools Console で以下を実行し結果を報告:
       - performance.getEntriesByType('navigation')[0]
       - performance.getEntriesByType('paint')
       - performance.getEntriesByType('resource').length
    4. スクリーンショットを撮影
  RecordingName: benchmark_data
```

### 2.1 Core Web Vitals 取得

```javascript
mcp_playwright_browser_evaluate:
  function: |
    () => {
      const nav = performance.getEntriesByType('navigation')[0];
      if (!nav) return JSON.stringify({ error: 'Navigation Timing unavailable (SPA soft navigation?). Re-run after a full page reload.' });
      const paint = performance.getEntriesByType('paint');
      const fcp = paint.find(e => e.name === 'first-contentful-paint');
      return JSON.stringify({
        ttfb_ms: Math.round(nav.responseStart - nav.requestStart),
        fcp_ms: fcp ? Math.round(fcp.startTime) : null,
        dom_interactive_ms: Math.round(nav.domInteractive - nav.startTime),
        dom_complete_ms: Math.round(nav.domComplete - nav.startTime),
        full_load_ms: Math.round(nav.loadEventEnd - nav.startTime),
        redirect_ms: Math.round(nav.redirectEnd - nav.redirectStart),
        dns_ms: Math.round(nav.domainLookupEnd - nav.domainLookupStart),
        tcp_ms: Math.round(nav.connectEnd - nav.connectStart)
      });
    }
```

### 2.1b LCP（Largest Contentful Paint）取得

LCP は `performance.getEntriesByType()` では取得できない。**PerformanceObserver** を使用:

```javascript
mcp_playwright_browser_evaluate:
  function: |
    async () => {
      return await new Promise((resolve) => {
        let lcpValue = null;
        const observer = new PerformanceObserver((list) => {
          const entries = list.getEntries();
          lcpValue = entries[entries.length - 1];
        });
        observer.observe({ type: 'largest-contentful-paint', buffered: true });
        setTimeout(() => {
          observer.disconnect();
          resolve(JSON.stringify({
            lcp_ms: lcpValue ? Math.round(lcpValue.startTime) : null,
            lcp_element: lcpValue ? lcpValue.element?.tagName : null,
            lcp_url: lcpValue?.url || null,
            lcp_size: lcpValue?.size || null
          }));
        }, 3000);
      });
    }
```

**注意**: LCPはページ操作なしで最後に報告された値が最終値。ページロード後すぐに取得すること。重いSPAではLCPイベントの発火が遅れる場合があるため、タイムアウトを3秒に設定。それでも取得できない場合はさらに延長を検討。

### 2.2 リソース分析

```javascript
mcp_playwright_browser_evaluate:
  function: |
    () => {
      const resources = performance.getEntriesByType('resource');
      const top15 = resources
        .map(r => ({
          name: r.name.split('/').pop().split('?')[0],
          type: r.initiatorType,
          size: r.transferSize,
          duration: Math.round(r.duration)
        }))
        .sort((a, b) => b.duration - a.duration)
        .slice(0, 15);
      return JSON.stringify(top15);
    }
```

### 2.3 バンドルサイズ分析

```javascript
mcp_playwright_browser_evaluate:
  function: |
    () => {
      const resources = performance.getEntriesByType('resource');
      const js = resources.filter(r => r.initiatorType === 'script')
        .map(r => ({ name: r.name.split('/').pop().split('?')[0], size: r.transferSize }));
      const css = resources.filter(r => r.initiatorType === 'css')
        .map(r => ({ name: r.name.split('/').pop().split('?')[0], size: r.transferSize }));
      return JSON.stringify({
        js_total: js.reduce((s, e) => s + (e.size || 0), 0),
        css_total: css.reduce((s, e) => s + (e.size || 0), 0),
        total_requests: resources.length,
        total_transfer: resources.reduce((s, e) => s + (e.transferSize || 0), 0),
        js_files: js,
        css_files: css
      });
    }
```

## Step 3: ベースライン保存（--baseline モード）

JSONファイルとして保存:

```json
{
  "url": "<url>",
  "timestamp": "<ISO>",
  "branch": "<branch>",
  "pages": {
    "/": {
      "ttfb_ms": 120,
      "fcp_ms": 450,
      "dom_interactive_ms": 600,
      "dom_complete_ms": 1200,
      "full_load_ms": 1400,
      "total_requests": 42,
      "total_transfer_bytes": 1250000,
      "js_bundle_bytes": 450000,
      "css_bundle_bytes": 85000,
      "largest_resources": []
    }
  }
}
```

保存先: `docs/benchmark-reports/baselines/baseline.json`

## Step 4: ベースライン比較

ベースラインが存在する場合、現在のメトリクスと比較:

```text
パフォーマンスレポート — [url]
══════════════════════════════════
ブランチ: [current] vs ベースライン ([baseline-branch])

メトリクス          ベースライン    現在        差分        ステータス
────────            ────────        ───────     ─────       ──────
TTFB                120ms           135ms       +15ms       OK
FCP                 450ms           480ms       +30ms       OK
LCP                 800ms           1600ms      +800ms      REGRESSION
DOM Interactive     600ms           650ms       +50ms       OK
DOM Complete        1200ms          1350ms      +150ms      WARNING
Total Transfer      1.2MB           1.8MB       +0.6MB      REGRESSION
JS Bundle           450KB           720KB       +270KB      REGRESSION
```

### 回帰検出しきい値

| メトリクス | WARNING | REGRESSION |
|-----------|---------|------------|
| タイミング | >20%増加 | >50%増加 OR >500ms絶対増加 |
| バンドルサイズ | >10%増加 | >25%増加 |
| リクエスト数 | >30%増加 | — |

## Step 5: パフォーマンスバジェットチェック

業界基準に対してチェック:

```text
パフォーマンスバジェット
════════════════════════
メトリクス       バジェット    実測        ステータス
────────         ──────       ──────      ──────
FCP              < 1.8s       0.48s       PASS
LCP              < 2.5s       1.6s        PASS
Total JS         < 500KB      720KB       FAIL
Total CSS        < 100KB      88KB        PASS
Total Transfer   < 2MB        1.8MB       WARNING (90%)
HTTP Requests    < 50         58          FAIL

グレード: B (4/6 合格)
```

## Step 6: トップ10遅延リソース

```text
TOP 10 遅延リソース
═════════════════════
#   リソース                  種別      サイズ     時間
1   vendor.chunk.js          script    320KB     480ms
2   main.js                  script    250KB     320ms
...

推奨:
- vendor.chunk.js: コード分割を検討 — 初期ロードに320KBは大きい
- analytics.js: async/defer で読み込み — 250msレンダリングをブロック
```

## Step 7: トレンド分析（--trend モード）

過去のベースラインファイルを読み込みトレンド表示:

```text
パフォーマンストレンド（過去5回）
════════════════════════════════
日付         FCP     LCP     Bundle    Requests    Grade
2026-03-10   420ms   750ms   380KB     38          A
2026-03-12   440ms   780ms   410KB     40          A
2026-03-14   450ms   800ms   450KB     42          A
2026-03-16   460ms   850ms   520KB     48          B
2026-03-18   480ms   1600ms  720KB     58          B

トレンド: パフォーマンス悪化中。LCPが8日で倍増。
         JSバンドルが週50KB成長。調査推奨。
```

## Step 8: レポート保存

```bash
run_command: mkdir -p docs/benchmark-reports
```

`docs/benchmark-reports/{date}-benchmark.md` と `.json` に保存。

## Step 9: Memory MCP に永続化

```yaml
mcp_memory_create_entities:
  - name: "[プロジェクト名]-benchmark-[日付]"
    entityType: "performance_benchmark"
    observations:
      - "URL: [対象URL]"
      - "Grade: [A/B/C/D/F]"
      - "REGRESSION: [検出された回帰の概要]"
      - "推奨: [主要な改善提案]"
```

## 完了レポート

```text
╔══════════════════════════════════════════╗
║  ULTRA-BENCHMARK 完了                    ║
║  STATUS: [DONE / DONE_WITH_CONCERNS]     ║
╠══════════════════════════════════════════╣
║  対象URL:   [URL]                        ║
║  グレード:  [A/B/C/D/F]                  ║
║  回帰検出:  [N] 件                       ║
║  WARNING:   [N] 件                       ║
║  推奨事項:  [N] 件                       ║
╚══════════════════════════════════════════╝
```

## 重要ルール

1. **推測ではなく計測** — `performance.getEntries()` の実データを使用
2. **ベースラインが不可欠** — ベースラインなしでは絶対値のみで回帰検出不可
3. **相対しきい値** — ダッシュボードの2000msは普通、LPなら問題。自分のベースラインと比較
4. **バンドルサイズが先行指標** — ロード時間はネットワーク依存、バンドルサイズは決定的
5. **読み取り専用** — レポート生成のみ。明示的に依頼されない限りコードは変更しない
