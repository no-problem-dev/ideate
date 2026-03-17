---
title: "Idea Mix Review"
created: 2026-03-18
version: 1.0.0
context: fork
model: claude-sonnet-4
tools: [Read, Glob, Grep]
tags: [skill, idea-mix, review]
status: active
type: skill
user_invocable: true
trigger: ["/idea-mix-review", "スパークレビュー", "アイデアレビュー"]
---

# Idea Mix Review

## CONTEXT

### 適用範囲

idea-mix で生成された過去の出力を振り返り、パターンを分析するスキル。
どの Lens 構成・合成モード・カード組み合わせが面白い結果を生んだかを可視化し、
今後の発想の質を向上させる。

### Diversity Engineering 依存

| DEP | 用途 |
|-----|------|
| DEP-1 | 過去出力の読み込みと傾向分析 |
| DEP-7 | novelty_score の推移追跡 |

## ROLE

| 属性 | 内容 |
|------|------|
| **専門性** | パターン分析、メタ認知支援、創造プロセスの最適化 |
| **権限** | 出力ファイルの読み取り、レビューレポートの生成 |
| **責任** | 発想プロセスの質的改善のための洞察提供 |

## INTENT

### Goal

過去の idea-mix セッションを振り返り、何が面白い概念を生むのかのパターンを発見する。

## STEPS

### 実行フロー

```
1. データ収集
   - Glob: ideas/idea-mix/*.md から全ファイルを取得
   - 各ファイルのフロントマターを解析:
     - collision.cards（使用カード）
     - collision.lens_frames（使用フレーム）
     - collision.synthesis_mode（合成モード）
     - novelty_score
     - status（raw/promising/prototyping/shipped）

2. パターン分析
   a. Lens 構成の効果
      - どの thinking_frame の組み合わせが高 novelty_score を出したか
      - 特に有効だったフレームの特定

   b. 合成モードの効果
      - emergent/bridge/invert のどれが面白い結果を出しやすいか
      - モード別の平均 novelty_score

   c. カード種別の効果
      - どの能力カードが最も多くの面白い概念に関わったか
      - 摩擦と刺激の組み合わせパターン

   d. ステータス推移
      - raw → promising → prototyping → shipped の変換率
      - 停滞している概念の特定

3. Novelty Health チェック
   - 過去4回の平均 novelty_score
   - N ≥ 4 の割合（目標: 40% 以上）
   - 低下トレンドがある場合はアラート

4. レポート生成
   Write: ideas/idea-mix/reviews/YYYY-MM-DD-review.md
```

### レポートフォーマット

```markdown
---
title: "Idea Mix Review"
created: YYYY-MM-DD
type: review
---

# Idea Mix Review — YYYY-MM-DD

## サマリー

- 総生成数: {count}
- 平均 novelty_score: {avg}
- ステータス分布: raw:{n} / promising:{n} / prototyping:{n} / shipped:{n}

## 効果的だったパターン

### Lens 構成
| フレーム組み合わせ | 使用回数 | 平均 N-score | 代表的な概念 |
|-------------------|---------|-------------|-------------|
| {frames} | {count} | {avg_n} | {concept_name} |

### 合成モード
| モード | 使用回数 | 平均 N-score | 特徴 |
|--------|---------|-------------|------|
| emergent | {n} | {avg} | {特徴} |
| bridge | {n} | {avg} | {特徴} |
| invert | {n} | {avg} | {特徴} |

### 能力カード活用度
| カード | 使用回数 | 関連概念数 | promising 以上 |
|--------|---------|----------|--------------|
| {card} | {n} | {n} | {n} |

## Novelty Health

- 直近4回平均: {avg}
- N ≥ 4 の割合: {pct}%
- トレンド: {上昇/安定/低下}
{低下の場合: ⚠️ モードコラプスの兆候。Lens 構成の変更を推奨。}

## 推奨アクション

- {アクション1}
- {アクション2}
- {アクション3}
```

## PROOF

### Do / Don't

**Do**:
- ✅ 定量的な分析に基づく推奨を行う
- ✅ novelty_score の推移を追跡する
- ✅ 停滞している概念を指摘する

**Don't**:
- ❌ 主観的な評価で終わる
- ❌ データが少ない段階で強い推奨をする
