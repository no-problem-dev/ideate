---
title: "Cognitive Isolation"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Cognitive Isolation

DEP-8（認知的隔離）の実装。複数の Lens エージェントに異なる部分情報と
異なる思考フレームを注入し、並列実行する。各 Lens は他の Lens の存在・入力・出力を知らない。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `cards` | カードの全集合（種別ごとに分類済み） | `{ capability: [...], friction: [...], stimulus: [...] }` |
| `lens_configs` | 各 Lens への設定配列 | 下記参照 |
| `output_format` | 結論の出力形式 | `"conclusion_only"` / `"with_reasoning"` |

### lens_configs の構造

```yaml
lens_configs:
  - lens_id: "deconstructor"
    input_filter:
      include: [friction]      # このカード種別のみ渡す
      exclude: [capability, stimulus]  # これらは隠蔽
    thinking_frame: "deconstruct"
    output_constraint: "結論のみ200字以内"

  - lens_id: "analogist"
    input_filter:
      include: [stimulus]
      exclude: [capability, friction]
    thinking_frame: "analogize"
    output_constraint: "結論のみ200字以内"

  - lens_id: "inverter"
    input_filter:
      include: [capability]
      exclude: [friction, stimulus]
    thinking_frame: "invert"
    output_constraint: "結論のみ200字以内"

  - lens_id: "constrainer"
    input_filter:
      include: [capability, friction]
      exclude: [stimulus]
    thinking_frame: "constrain"
    output_constraint: "結論のみ200字以内"
```

## Steps

### Step 1: 各 Lens のプロンプト構築

```
FOR EACH config IN lens_configs:

  1. input_filter に基づき、cards から該当するカードを抽出
     - include に指定された種別のカードのみ選択
     - exclude に指定された種別は完全に除外
     - カードの出自（capability/friction/stimulus）は伝えても良い

  2. プロンプトを構築:
     ---
     あなたには以下の情報だけが与えられています。
     これが全てです。他に情報はありません。

     {抽出されたカードを列挙}

     上記の情報に基づき、「{thinking_frame}」の思考法で分析してください。

     思考法の説明:
     {lens.md の組み込み思考フレームから該当する説明}

     出力制約: {output_constraint}
     ---

  3. 重要: 以下を含めない
     - 他の Lens の存在
     - 隠蔽された種別のカード
     - 最終的な合成目的
```

### Step 2: 並列実行

```
全 lens_configs の Lens を Agent tool で同時に起動:

FOR EACH config IN lens_configs:
  Agent(
    name: "lens-{config.lens_id}",
    model: sonnet,
    prompt: Step 1 で構築したプロンプト
  )

全 Agent の完了を待機。
```

### Step 3: 結論の収集

```
各 Lens の出力を収集:

IF output_format == "conclusion_only":
  各 Lens の出力から結論部分のみを抽出
  推論過程が含まれていた場合は破棄

IF output_format == "with_reasoning":
  推論過程も含めて収集（合成者に渡す情報量が増える）

結論を配列として返す:
  conclusions:
    - lens_id: "deconstructor"
      frame: "deconstruct"
      conclusion: "{結論テキスト}"
    - lens_id: "analogist"
      frame: "analogize"
      conclusion: "{結論テキスト}"
    ...
```

## Output

| 出力 | 説明 |
|------|------|
| `conclusions` | 各 Lens の結論配列（lens_id, frame, conclusion） |
| `lens_count` | 実行された Lens の数 |
| `isolation_metadata` | 各 Lens に渡された情報の要約（監査用） |

## カスタマイズ

### Lens 構成のオーバーライド

スキルは `lens_configs` を自由にカスタマイズできる。

```yaml
# 例: 5つの Lens で異なる粒度の情報を渡す
lens_configs:
  - lens_id: "micro"
    input_filter: { include: [friction] }  # 摩擦だけ
    thinking_frame: "deconstruct"

  - lens_id: "macro"
    input_filter: { include: [stimulus] }  # 刺激だけ
    thinking_frame: "scale"

  - lens_id: "hands"
    input_filter: { include: [capability] }  # 能力だけ
    thinking_frame: "invert"

  - lens_id: "bridge"
    input_filter: { include: [capability, friction] }  # 能力+摩擦
    thinking_frame: "constrain"

  - lens_id: "wild"
    input_filter: { include: [stimulus, friction] }  # 刺激+摩擦
    thinking_frame: "reframe"
```

### 情報遮断の強度レベル

| レベル | 入力 | フレーム | 推論過程 | 用途 |
|--------|------|---------|---------|------|
| **Full** | 部分情報 | 単一フレーム強制 | 結論のみ | 最大の多様性が必要な場合 |
| **Medium** | 部分情報 | 単一フレーム強制 | 推論過程も伝搬 | 合成者に文脈を与えたい場合 |
| **Light** | 全情報 | 異なるフレーム | 結論のみ | フレーム多様性のみ確保 |

## コスト見積もり

- Lens 4 個 × Sonnet = 4 回の Agent 呼び出し
- 並列実行のため時間コストは 1 回分
- 単発実行スキルでの使用を想定、月 4-8 回程度
