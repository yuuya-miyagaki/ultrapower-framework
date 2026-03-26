# Ultra-Implement — Learned Patterns

> Auto-Evolve / ultra-retro からの学習パターン蓄積。実装時に自動参照される。

---

### Firestore 再帰的サブコレクション（自動検出: 2026-03-25）

**プロジェクト**: Small World MVP
**状況**: Firestore で深いネスト構造（`worlds/{id}/agents/{id}/shortTermMemories/{id}`）を使用した際、セキュリティルールで個別パスを列挙していたためサブコレクションの権限が漏れた。
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

**プロジェクト**: Small World MVP
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
