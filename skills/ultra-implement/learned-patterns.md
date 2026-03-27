# Ultra-Implement — Learned Patterns

> Auto-Evolve / ultra-retro からの学習パターン蓄積。実装時に自動参照される。

---

### Firestore 再帰的サブコレクション（自動検出: 2026-03-25）

**状況**: Firestore で深いネスト構造（3階層以上のサブコレクション）を使用した際、セキュリティルールで個別パスを列挙していたためサブコレクションの権限が漏れた。
**解決策**: `match /{subPath=**}` で全サブコレクションを一括カバー。

```text
// ✅ 推奨: 再帰的ワイルドカード（漏れなし）
match /worlds/{worldId} {
  match /{subPath=**} {
    allow read, write: if isWorldOwner(worldId);
  }
}

// ❌ 非推奨: 個別パス列挙（漏れが発生しやすい）
match /worlds/{worldId}/agents/{agentId} { ... }
match /worlds/{worldId}/channels/{channelId} { ... }
// agents/{agentId}/shortTermMemories が漏れた!
```

**適用条件**: 3階層以上のサブコレクション構造を持つ Firebase プロジェクト。

---

### HF Inference API リトライパターン（自動検出: 2026-03-25）

**状況**: Hugging Face Inference API が断続的に 400/503 エラーを返す。
**解決策**: 指数バックオフ付きリトライ（最大3回）を実装。

```javascript
async function callWithRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === maxRetries - 1) throw e;
      await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
    }
  }
}
```

**適用条件**: 外部 AI/ML API を呼び出すプロジェクト全般。

---

### 定数は共通モジュールに集約（自動検出: 2026-03-27）

**プロジェクト**: Small World
**状況**: 動的 `import()` でコード分割した際、`MAX_AGENTS` 定数が2ファイルに複製された。一方を変更しても他方が追従せず技術的負債に。
**解決策**: `config/constants.js` に一元定義し、各モジュールからインポート。

```javascript
// ✅ 推奨: 共通定数モジュール
// config/constants.js
export const MAX_AGENTS = 6;

// 各ファイルから import
import { MAX_AGENTS } from '../../config/constants.js';

// ❌ 非推奨: 同じ値を複数ファイルに定義
const MAX_AGENTS = 6; // ← dashboard.js にも agentCreator.js にも
```

**適用条件**: 2つ以上のモジュールが同じマジックナンバーを参照する場合。

---

### バニラJS の手動双方向バインディング（自動検出: 2026-03-27）

**プロジェクト**: Small World
**状況**: React/Vue 非使用のバニラJSプロジェクトで、フォーム状態とDOM表示の同期が必要になった。
**解決策**: `formState` オブジェクト + `applyStateToDOM()` 関数のペアで明示的に同期。

```javascript
// 状態オブジェクト（Single Source of Truth）
const formState = { name: '', personality: { openness: 0.5 } };

// DOM → state: イベントリスナーで更新
input.addEventListener('input', (e) => { formState.name = e.target.value; });

// state → DOM: 明示的に反映（プリセット適用時等）
function applyStateToDOM(state) {
  document.getElementById('name').value = state.name;
  slider.value = state.personality.openness;
}
```

**適用条件**: バニラJS でフォーム UI を構築する場合。フレームワーク導入が不可/過剰な場合に有効。

---

### テンプレートから生成したデータのフラグ固定（自動検出: 2026-03-27）

**プロジェクト**: Small World
**状況**: プリセット（テンプレート）からエージェントを作成した際、`isPreset: true` がそのまま引き継がれ、ユーザー作成エージェントが削除不可になった。
**解決策**: `buildData()` 関数で、テンプレート由来のフラグを明示的にオーバーライド。

```javascript
// ✅ 推奨: テンプレートのフラグを上書き
function buildAgentData(formState) {
  return {
    ...formState,
    isPreset: false,  // ← ユーザー作成は常にカスタム
    createdAt: new Date(),
  };
}

// ❌ 非推奨: テンプレートのフラグをそのまま渡す
function buildAgentData(formState) {
  return { ...formState };  // isPreset: true が紛れ込む
}
```

**適用条件**: テンプレート/プリセットからデータを生成する機能全般。

