---
title: "Parallel Critique Pipeline"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Parallel Critique Pipeline

DEP-4（Critic Diversity）の実装。提案に対して複数の異なる観点から
並列に批判を行い、結果を統合して robust な最終出力を生成する。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `proposals` | 批判対象の提案リスト | Top 5 アイデア |
| `critique_frames` | 批判の観点リスト（2-4 個推奨） | 下記参照 |
| `survival_threshold` | 生存に必要な最低通過フレーム数 | `2`（3 中 2 通過で生存） |

## Steps

### Step 1: 批判フレームの選定

デフォルトの批判フレーム（スキルがオーバーライド可能）:

```yaml
critique_frames:
  - name: "技術的実現性"
    agent: critic
    prompt: |
      以下の提案について、技術的実現性の観点から批判してください。
      - 実装に必要な技術は現実的か
      - 個人開発者が 3 ヶ月以内に MVP を作れるか
      - 技術的な落とし穴やリスクは何か
      判定: PASS（実現可能）/ FAIL（実現困難）+ 根拠

  - name: "ユーザー心理"
    agent: critic
    prompt: |
      以下の提案について、ユーザー心理の観点から批判してください。
      - ユーザーは本当にこの問題を解決したいと思っているか
      - 既存の習慣やワークアラウンドを捨ててまで使うか
      - 継続利用のモチベーションは何か
      判定: PASS（使われる）/ FAIL（使われない）+ 根拠

  - name: "ビジネス成立性"
    agent: critic
    prompt: |
      以下の提案について、ビジネスの観点から批判してください。
      - 収益化の道筋はあるか
      - 市場規模は個人開発で十分か
      - 競合に対する持続的な優位性はあるか
      判定: PASS（成立する）/ FAIL（成立しない）+ 根拠
```

### Step 2: 並列批判の実行

```
各 critique_frame について Agent tool で並列に Task を起動:

FOR EACH proposal IN proposals:
  FOR EACH frame IN critique_frames:
    Task(model: sonnet, agent: critic):
      「{frame.prompt}

       対象提案:
       {proposal の内容}

       判定と根拠を 200 文字以内で簡潔に回答してください。」

全 Task の完了を待機。
```

### Step 3: 結果の集約

```
各 proposal について:
  pass_count = PASS 判定の数
  fail_count = FAIL 判定の数

  IF pass_count >= survival_threshold:
    status = "SURVIVED"
    strength_note = PASS 判定の根拠を統合
    weakness_note = FAIL 判定の根拠を統合（改善ポイントとして記録）
  ELSE:
    status = "ELIMINATED"
    elimination_reason = FAIL 判定の根拠を統合
```

### Step 4: 統合レポートの生成

```markdown
## Critique Results

| # | Proposal | 技術 | 心理 | ビジネス | 判定 |
|---|----------|------|------|---------|------|
| 1 | {title}  | PASS | PASS | FAIL    | SURVIVED (2/3) |
| 2 | {title}  | FAIL | FAIL | PASS    | ELIMINATED (1/3) |
...

### Survived Proposals
{各 SURVIVED 提案の強み + 改善ポイント}

### Key Insights from Eliminated Proposals
{ELIMINATED 提案から学べる共通の弱点パターン}
```

## Output

| 出力 | 説明 |
|------|------|
| `survived_proposals` | 批判を生き延びた提案リスト + 強み/弱みの注釈 |
| `eliminated_proposals` | 排除された提案 + 排除理由 |
| `critique_summary` | 統合レポートテキスト |

## コスト見積もり

- 提案 5 件 × 批判フレーム 3 個 = 15 回の Agent 呼び出し（Sonnet）
- 週 1 回実行の場合、月 60 回。コスト許容範囲内。
