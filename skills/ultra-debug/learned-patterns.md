# Ultra-Debug — Learned Patterns

> Auto-Evolve / ultra-retro からの学習パターン蓄積。デバッグ時に自動参照される。

---

### 推測より観察（Observe Before Guessing）（自動検出: 2026-03-28）

**プロジェクト**: Small World（マルチエージェントAIプラットフォーム）
**状況**: エージェントの会話品質が壊滅的で、定型文しか返さなかった。コードを読んでプロンプト構造・会話履歴構築ロジック・システムプロンプトの改善に数時間を費やしたが、根本原因は「APIキーのクォータがゼロ（`limit: 0`）でLLMが一度も応答を生成していなかった」こと。
**解決策**: デバッグの最初のステップは**まず動かしてコンソールを見る**。ブラウザ/ターミナルでの実行結果 → コンソールログ → ネットワークリクエストの順で確認。

```text
# デバッグの優先順位
1. まず動かす（ブラウザを開く or コマンドを実行する）
2. エラーを観察する（コンソール、ネットワーク、ステータスコード）
3. 仮説を立てる（観察結果に基づいて）
4. コードを読む（仮説を検証するために）

# ❌ やってはいけない順序
1. コードを読む
2. 推測で問題を特定する
3. コードを修正する
4. 動かして確認する（ここで初めてAPIが動いてないことに気づく）
```

**適用条件**: 全てのデバッグ。特にAPI連携・外部サービス依存のプロジェクト。

---

### 外部依存の疎通確認を最初にやる（Verify External Dependencies First）（自動検出: 2026-03-28）

**プロジェクト**: Small World
**状況**: Gemini API を使う3つのエージェントが全て定型文を返していた。コード側のプロンプト改善をいくら行っても、APIが `429 RESOURCE_EXHAUSTED (limit: 0)` を返しているため効果がなかった。
**解決策**: デバッグ開始時に外部依存の疎通を最初に確認する。

```bash
# API疎通確認の定型手順

# 1. APIキーの有効性テスト（curl で直接叩く）
curl -s "https://API_ENDPOINT?key=API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"minimal": "request"}' | python3 -m json.tool

# 2. 確認すべき項目
#    - ステータスコード（200? 401? 403? 429?）
#    - クォータ/レート制限（limit, remaining）
#    - 認証エラー（invalid key, expired token）
#    - エンドポイント/モデル名の正確さ（404）

# 3. フォールバック戦略
#    - 別プロバイダへの切り替え（Gemini → HF Inference）
#    - モデルの変更（より制限の緩いモデルへ）
#    - 1行フラグで切替可能な設計にする
```

```javascript
// ✅ 推奨: 1行フラグでプロバイダ切替可能
const useHf = true; // ← false にすれば Gemini に戻る
if (useHf || provider === 'huggingface') {
  return hfChat(messages, options);
}
return geminiChat(messages, options);
```

**適用条件**: 外部API（AI/ML API、決済、認証サービス等）に依存するプロジェクト全般。

---

### バックグラウンドプロセスのリソース消費（Background Process Resource Drain）（自動検出: 2026-03-28）

**プロジェクト**: Small World
**状況**: 自律発言機能（ハートビート）が5分ごとに全エージェント分のAPIを呼び出し。1ハートビートで `decide()` + `executeAction()` = 2回のAPI呼び出し × 3エージェント = 6リクエスト/5分。これがGemini APIの無料枠（日次20リクエスト）を1時間以内に枯渇させていた。APIエラー時にフォールバック定型文（「進捗どうですか？」）がチャットに投稿され続け、UIもノイズまみれに。
**解決策**: 開発中はバックグラウンドプロセスを完全停止。関数の中身を空にして呼び出し元を変更せずに無効化。

```javascript
// ✅ 推奨: 関数を空にして安全に無効化（呼び出し元の変更不要）
function setupHeartbeats(state) {
  console.log('[Autonomy] Heartbeat disabled — manual chat only');
}

// ❌ 非推奨: 呼び出し箇所をコメントアウト（漏れが発生しやすい）
// setupHeartbeats(state);  // ← 他の箇所でも呼ばれている可能性
```

**チェックリスト（API依存プロジェクト）:**

```text
□ setInterval / setTimeout の再帰呼び出しを検索
□ cron / scheduler / heartbeat / polling の存在確認
□ 各バックグラウンドプロセスのAPI呼び出し回数を計算
□ 合計を無料枠と比較（例: 6req/5min × 12 = 72req/hour > 20req/day）
□ 開発中は全て停止し、ユーザー操作時のみAPI呼び出し
```

**適用条件**: APIクォータに制限があるプロジェクト（特にAI/ML API無料枠使用時）。

---

### LLMメッセージ配列の構造検証（LLM Message Array Structure Validation）（自動検出: 2026-03-28）

**プロジェクト**: Small World
**状況**: マルチエージェント環境で、他エージェントの発言も全て `role: 'user'` でLLMに送信。Gemini APIは連続する同一ロールを結合するため、ユーザーの質問と他エージェントの発言が1つのメッセージにマージされ、文脈が崩壊。さらに `incomingMessage` が記憶と重複して同じ質問が2回送信されていた。
**解決策**: LLMに送る前にメッセージ配列を検証・正規化する。

```javascript
// LLMメッセージ配列の正規化チェックリスト:
//
// 1. role の分類が正しいか
//    - 自分以外の発言 → 'user'（LLMから見た入力）
//    - 自分の過去の発言 → 'model'/'assistant'
//    - 同じ内容が2つの異なるroleで送られていないか
//
// 2. 連続する同一ロールがないか
//    - Gemini: 連続 user を自動マージ → 意図しない結合
//    - OpenAI互換: 連続 user は許容されるが冗長
//    → 連続する同一ロールは1つに結合
//
// 3. メッセージの重複がないか
//    - 記憶から取得したメッセージと新規メッセージの重複
//    - filter() で content が一致するものを除外

// ✅ 推奨: デバッグログを仕込んでおく
console.log('[LLM] Messages to send:', JSON.stringify(messages.map(m => ({
  role: m.role,
  content: m.content.substring(0, 50) + '...'
}))));
```

**適用条件**: LLM API を使用する全てのプロジェクト。特にマルチエージェント・チャット履歴が関わる場合。
