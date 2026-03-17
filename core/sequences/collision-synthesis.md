---
title: "Collision Synthesis"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Collision Synthesis

DEP-9（衝突合成）の実装。DEP-8 で生成された独立した結論群を衝突させ、
どの単一視点からも到達できなかった新概念を生成する。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `conclusions` | DEP-8 の出力（結論の配列） | cognitive-isolation の出力 |
| `synthesis_mode` | 合成モード | `"emergent"` / `"bridge"` / `"invert"` |
| `concept_count` | 生成する概念数 | `3`〜`5` |
| `novelty_constraint` | DEP-1 の出力（過去出力との差分情報。省略可） | diversity-context-loader の出力 |

## Steps

### Step 1: 結論の提示

```
以下の結論群を合成者に提示する。
重要: 推論過程は含めない。結論のみを並べる。

---
以下は、互いの存在を知らない独立した視点から生成された結論です。
各視点は異なる部分情報と異なる思考法のみに基づいています。

結論 A ({conclusions[0].frame}):
  {conclusions[0].conclusion}

結論 B ({conclusions[1].frame}):
  {conclusions[1].conclusion}

結論 C ({conclusions[2].frame}):
  {conclusions[2].conclusion}

結論 D ({conclusions[3].frame}):
  {conclusions[3].conclusion}
---
```

### Step 2: 合成モードに応じた操作

```
[synthesis_mode = "emergent"]
プロンプト:
「上記の結論群の中で、矛盾する点・緊張関係にある点を特定してください。
 その矛盾が『両方正しい』ような上位概念を {concept_count} 個生成してください。
 矛盾の解消ではなく、矛盾を包含する新しい概念を目指してください。」

[synthesis_mode = "bridge"]
プロンプト:
「上記の結論群は表面的には無関係に見えます。
 しかし、これらが同時に生成された背景には隠れた共通構造があるはずです。
 その隠れた構造を {concept_count} 個発見し、概念化してください。
 『なぜこれらの結論が同時に存在しうるのか』を問うてください。」

[synthesis_mode = "invert"]
プロンプト:
「上記の全ての結論が暗黙に前提としていること（共通の盲点）を特定してください。
 その前提を反転させたとき、どんな概念が生まれますか？
 {concept_count} 個の反転概念を生成してください。
 全員が見落としている盲点を概念化してください。」
```

### Step 3: 新規性チェック

```
各生成概念に対して:

1. 「これは既存の〜と同じではないか？」を自問
   - 既知のサービス、フレームワーク、概念との照合
   - 同定できた場合は却下し、代替を生成

2. 「なぜこれは今まで存在しなかったか？」を説明
   - 説明がつかない（= 既に存在する可能性が高い）場合は却下

3. 「最小の実験で検証するには？」を提案
   - 1日以内でできる検証方法を考える

4. novelty_constraint が提供されている場合:
   - 過去出力との重複チェックも実施
   - DEP-7 の基準で N スコアを付与
```

### Step 4: 出力の構成

```
生成された概念を以下の形式で構造化:

concepts:
  - name: "{概念名}"
    description: "{3文以内の説明}"
    collision_source: "結論{A}×結論{B}"
    novelty_reason: "{なぜ新しいか}"
    minimum_experiment: "{最小検証方法}"
    novelty_score: {1-5}  # DEP-7 基準

rejected:
  - name: "{却下された概念名}"
    reason: "{却下理由}"
```

## Output

| 出力 | 説明 |
|------|------|
| `concepts` | 新概念の配列（各概念に name, description, collision_source, novelty_reason, minimum_experiment, novelty_score） |
| `rejected` | 却下された概念と理由 |
| `synthesis_mode` | 使用された合成モード |

## 合成モードのローテーション

繰り返し実行する場合、synthesis_mode を週単位でローテーションさせる:

```
ISO 週番号 % 3:
  0 → emergent（矛盾からの創発）
  1 → bridge（隠れた共通構造）
  2 → invert（暗黙の前提の反転）
```

## 使用例

```markdown
# idea-mix スキルでの使用例

## Phase 4: 衝突合成

collision-synthesis を以下で実行:
  conclusions: Phase 3 の cognitive-isolation 出力
  synthesis_mode: "emergent"（今週のローテーション）
  concept_count: 3
  novelty_constraint: Step 0 で生成した過去出力制約（あれば）

→ 3 つの新概念が生成され、ユーザーに提示される
```
