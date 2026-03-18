---
title: "Cognitive Isolation"
created: 2026-03-18
version: 2.0.0
parent: core
type: sequence
---

# Cognitive Isolation

DEP-8（認知的隔離）の実装。複数の Lens エージェントに異なる部分情報と
異なる思考フレームを注入し、並列実行する。

v2 の変更点:
- Lens エージェントが多段推論を行う（1ターン完結ではない）
- 入力が「原理」の形式を取る（principle-extractor の出力）
- 各 Lens はツールを使った調査・自問自答が可能

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `principles` | 能力カードから抽出された成功原理群 | principle-extractor の出力 |
| `frictions` | 摩擦カード群 | ideas/idea-mix/deck/frictions.md の該当エントリ |
| `stimuli` | 刺激カード群 | ideas/idea-mix/deck/stimuli.md の該当エントリ |
| `lens_configs` | 各 Lens への設定配列 | 下記参照 |

### lens_configs の構造（v2: 原理ベース）

```yaml
lens_configs:
  - lens_id: "principle-friction"
    input:
      type: principle_and_friction
      principle: "{principles から1つ}"
      frictions: "{摩擦カード5-10枚}"
    thinking_frame: "constrain"
    instruction: |
      この原理を使って、以下の摩擦のうち構造的に噛み合うものを見つけ、
      原理の転写による解決策を深く考えてください。
      噛み合わないものは無理に接続せず、噛み合うものだけを深掘りしてください。

  - lens_id: "principle-stimulus"
    input:
      type: principle_and_stimulus
      principle: "{principles から1つ（別の原理）}"
      stimuli: "{刺激カード5-10枚}"
    thinking_frame: "analogize"
    instruction: |
      この原理と構造的に類似するパターンを刺激カードの中から見つけ、
      類推による新しい応用を深く考えてください。

  - lens_id: "friction-deep"
    input:
      type: friction_only
      frictions: "{摩擦カード3-5枚}"
    thinking_frame: "deconstruct"
    instruction: |
      これらの摩擦の根本原因を徹底的に掘り下げてください。
      表面的な原因ではなく、構造的・本質的な原因に到達してください。
      能力カードや解決策は一切考えないでください。

  - lens_id: "stimulus-reframe"
    input:
      type: stimulus_only
      stimuli: "{刺激カード3-5枚}"
    thinking_frame: "reframe"
    instruction: |
      これらの刺激が示すパターンから、「普通は問わない問い」を立ててください。
      問題の定義自体を書き換えるような視点を探してください。
```

## Steps

### Step 1: Lens へのインプット構築

```
各 lens_config に基づき、Lens エージェントへの指示を構築する。

重要: 各 Lens は以下を知らない:
  - 他の Lens の存在
  - 自分に渡されていない種別の情報
  - 最終的に合成されるという目的
  - 能力カード全体（principle-friction Lens には原理のみ、能力カードの全文は渡さない）

各 Lens の指示には以下を含める:
  - partial_input: 該当する部分情報のみ
  - thinking_frame: 思考フレームの指定
  - instruction: Lens 固有の指示
  - 「多段推論を行うこと」の明示（1ターンで結論を出さないこと）
  - 「ツール（Read, WebSearch 等）を積極的に使うこと」の明示
```

### Step 2: 並列実行

```
全 Lens を Agent tool で同時に起動:

FOR EACH config IN lens_configs:
  Agent(
    name: "lens-{config.lens_id}",
    model: sonnet,
    prompt: """
      あなたは独立した分析者です。以下の情報だけが与えられています。
      他に情報はありません。他の分析者もいません。

      {config.input の内容}

      上記の情報に基づき、以下の思考法で深く分析してください:
      思考法: {config.thinking_frame}

      {config.instruction}

      重要:
      - 1ターンで結論を出さないでください
      - まず情報を深く理解し、仮説を立て、自分の仮説を批判し、精製してください
      - 必要に応じて WebSearch で外部情報を調査してください
      - 最終的に以下の形式で出力してください:

      **洞察**: {核心的な洞察。1-2文}
      **到達経路**: {どういう思考でここに到達したか。3文以内}
      **開く可能性**: {この洞察が正しいとすると他に何が言えるか。2-3個}
    """
  )

全 Agent の完了を待機。
```

### Step 3: 洞察の収集

```
各 Lens の出力を収集:
  - 「洞察」「到達経路」「開く可能性」の3要素を構造化
  - 推論の詳細過程は合成者に渡さない（洞察の3要素のみ）

insights:
  - lens_id: "principle-friction"
    frame: "constrain"
    insight: "{洞察テキスト}"
    path: "{到達経路}"
    possibilities: ["{可能性1}", "{可能性2}", "{可能性3}"]
  - lens_id: "principle-stimulus"
    ...
```

## Output

| 出力 | 説明 |
|------|------|
| `insights` | 各 Lens の洞察配列（lens_id, frame, insight, path, possibilities） |
| `lens_count` | 実行された Lens の数 |
| `isolation_metadata` | 各 Lens に渡された情報の要約（監査用） |
