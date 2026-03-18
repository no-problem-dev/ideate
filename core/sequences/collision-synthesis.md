---
title: "Collision Synthesis"
created: 2026-03-18
version: 2.0.0
parent: core
type: sequence
---

# Collision Synthesis

DEP-9（衝突合成）の実装。DEP-8 で生成された独立した洞察群を衝突させ、
どの単一視点からも到達できなかった新概念を生成する。

v2 の変更点:
- collision-synthesizer エージェントが多段推論を行う
- WebSearch による既存性チェックが必須
- 洞察の「開く可能性」を手がかりに接続点を探す

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `insights` | DEP-8 の出力（洞察の配列） | cognitive-isolation v2 の出力 |
| `synthesis_mode` | 合成モード | `"emergent"` / `"bridge"` / `"invert"` |
| `concept_count` | 生成する概念数 | `3`〜`5` |
| `novelty_constraint` | DEP-1 の出力（省略可） | diversity-context-loader の出力 |

## Steps

### Step 1: collision-synthesizer エージェントを起動

```
Agent(
  name: "collision-synthesizer",
  model: opus,  ← 深い合成推論が必要なため Opus
  prompt: """
    あなたは独立した複数の分析者が生成した洞察を衝突させ、
    どの分析者も単独では到達できなかった新概念を生み出す合成者です。

    以下の洞察は、互いの存在を知らない独立した視点から生成されました。
    各洞察は異なる部分情報と異なる思考法に基づいています。

    {FOR EACH insight IN insights:}
    ---
    洞察 {index} ({insight.frame}):
      核心: {insight.insight}
      到達経路: {insight.path}
      開く可能性: {insight.possibilities}
    ---

    合成モード: {synthesis_mode}
    生成する概念数: {concept_count}

    {novelty_constraint があれば:}
    ## 過去出力との重複回避
    {novelty_constraint}

    上記の洞察を衝突させ、新概念を生成してください。
    collision-synthesizer.md のプロトコルに従い、多段階の思考を行ってください。

    重要:
    - 1ターンで結論を出さないでください
    - 洞察間の矛盾・共通構造・暗黙の前提を丁寧に分析してください
    - 必ず WebSearch で新規性をチェックしてください
    - 折衷案や要約は概念ではありません
  """
)
```

### Step 2: 出力の受け取り

```
collision-synthesizer が多段推論の末に返す出力を受け取る。
出力形式は collision-synthesizer.md の出力フォーマットに従う。
```

## Output

| 出力 | 説明 |
|------|------|
| `concepts` | 新概念の配列 |
| `rejected` | 却下された概念と理由 |
| `synthesis_mode` | 使用された合成モード |

## 合成モードのローテーション

```
ISO 週番号 % 3:
  0 → emergent（矛盾からの創発）
  1 → bridge（隠れた共通構造）
  2 → invert（暗黙の前提の反転）
```
