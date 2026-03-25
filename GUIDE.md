# Ultrapower クイックリファレンス

> 各スキルの詳細プロセスは `skills/SKILL_NAME/SKILL.md` を参照。

## スキル一覧

| スキル | トリガーワード | 引数 |
| ------ | ------------- | ---- |
| **ultrapower-workflow** | 自然言語での開発要望 | — |
| **ultra-onboard** | 「コード読んで」「理解して」 | — |
| **ultra-brainstorm** | 「アイデアを考えたい」「作りたい」 | — |
| **ultra-design-system** | 「デザインを決めよう」 | — |
| **ultra-plan** | 「計画を立てて」 | `--quick`, `--light` |
| **ultra-implement** | 計画承認後 / 「実装開始」 | — |
| **ultra-review** | 「レビューして」 | — |
| **ultra-qa** | 「テストして」 | — |
| **ultra-benchmark** | 「パフォーマンスは？」 | `--baseline`, `--compare` |
| **ultra-design-review** | 「デザインを確認して」 | — |
| **ultra-ship** | 「デプロイして」 | — |
| **ultra-debug** | バグ発見時 / 「バグを直して」 | — |
| **ultra-retro** | 「振り返りしよう」 | `24h`, `14d`, `30d`, `compare`, `global`, `auto` |
| **ultra-docs** | 「ドキュメント更新して」 | — |
| **ultra-second-opinion** | 「別の意見が欲しい」 | — |
| **ultra-parallel** | 「並列で」「同時に」 | — |

## 典型フロー

```text
onboard → brainstorm → plan → implement → review → qa → ship → retro
                                  ↑                       |
                                  └──── debug ←───────────┘
```

## 安全ガードレール

| ガードレール | 内容 |
|------------|------|
| **See Something Say Something** | 問題を発見したら即報告 |
| **3回ルール** | 同じ問題に3回失敗 → エスカレーション |
| **TDD厳守** | テストなしに本番コードを書かない（例外: CSS/設定/ドキュメント） |
| **検証なき完了禁止** | テスト実行せずに完了宣言しない |
| **Boil the Lake** | 部分的な実装で妥協しない |

## Tips

- 大規模PJでは `ultra-plan --light` で素早く計画
- DB自動検出: Supabase/Firebase を `.env` や設定ファイルから自動判定
- Memory MCP: `ultra-onboard` と `ultra-retro` で知識を永続化
- Context7: `ultra-plan` と `ultra-implement` でライブラリドキュメントを自動参照
- Hugging Face: `InferenceClient` でチャット・要約・翻訳・画像生成等のAIモデルを活用
