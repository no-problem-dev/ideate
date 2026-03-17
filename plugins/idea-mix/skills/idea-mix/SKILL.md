---
title: "Idea Mix"
created: 2026-03-18
version: 1.0.0
context: main
model: claude-opus-4
tools: [Read, Write, Edit, Bash, Grep, Glob, Agent, WebSearch]
tags: [skill, idea-mix, ideation, diversity-engineering]
status: active
type: skill
user_invocable: true
trigger: ["/idea-mix", "アイデアミックス", "発想", "ひらめき", "アイデア出し"]
---

# Idea Mix

## CONTEXT

### 適用範囲

能力・摩擦・刺激の3種類のカードを認知的隔離（DEP-8）と衝突合成（DEP-9）で
組み合わせ、まだ世に存在しない概念を生成する汎用アイデア発想スキル。

アプリ・サービスに限定せず、あらゆるドメインの「こんなことできるんじゃないか」を
生み出すことを目的とする。

### 用語

| 用語 | 定義 |
|------|------|
| 能力カード | ユーザーが実際に作れる/できることの抽象的記述 |
| 摩擦カード | 日常で感じる不便・非効率・構造的問題の記述 |
| 刺激カード | 他領域の面白い構造・パターン・トレンド |
| Lens | 部分情報+単一思考フレームで推論する隔離エージェント |
| 衝突合成 | 隔離されたLensの結論を衝突させ新概念を生む操作 |

### 前提条件

- `ideas/idea-mix/deck/capabilities.md` が存在すること（能力カード）
- `ideas/pain-log/` にエントリがあること（摩擦カードのソース、なくても可）
- `ideas/`, `knowledge/` 配下にTIL・トレンド等があること（刺激カードのソース、なくても可）

### Diversity Engineering 依存

| DEP | 用途 | 実装 |
|-----|------|------|
| DEP-1 | 過去の出力との重複回避 | diversity-context-loader |
| DEP-2 | Lens構成・合成モードのローテーション | rotation-engine |
| DEP-7 | 新規性スコアの付与 | collision-synthesis 内 |
| DEP-8 | Lensの認知的隔離 | cognitive-isolation |
| DEP-9 | 結論の衝突合成 | collision-synthesis |

## ROLE

| 属性 | 内容 |
|------|------|
| **専門性** | 創造的発想のオーケストレーション、認知的多様性の構造的確保 |
| **権限** | カードの選択、Lens構成の決定、合成結果の記録 |
| **責任** | 既存のどの視点からも到達できない新概念の生成 |
| **禁止事項** | ネット上の既存アイデアの集約で終わること |

## INTENT

### Goal

ユーザーの手持ちのカード（能力・摩擦・刺激）を構造的に衝突させ、
まだ言語化されていない概念を生成し、対話を通じて研ぎ澄ませる。

### Value

- **V-1**: 自分では思いつかなかった概念への到達
- **V-2**: 異なる視点の強制的な衝突による認知的飛躍
- **V-3**: 対話による概念の研磨と実験可能な形への着地

### Success Criteria

- [ ] 生成された概念の少なくとも1つが「既存のものと明確に異なる」
- [ ] ユーザーが「これは面白い」と感じる概念が含まれる
- [ ] 概念が最小実験に落とし込まれている

## STEPS

### 実行フロー

```
Phase 1: デッキ準備 (自動)
    ↓
Phase 2: カード選択 + 提示 (ユーザー確認可)
    ↓
Phase 3: 認知的隔離実行 (DEP-8, 並列・自動)
    ↓
Phase 4: 衝突合成 (DEP-9, 自動)
    ↓
Phase 5: 対話的深掘り (ユーザーと対話)
    ↓
Phase 6: 記録 (自動)
```

---

### Phase 1: デッキ準備

```
1-1. 能力カードの読み込み
     - Read: ideas/idea-mix/deck/capabilities.md
     - 各カードの abstract 表現を使用

1-2. 摩擦カードの収集
     - Glob: ideas/pain-log/ から直近2週間のエントリを取得
     - 高頻度カテゴリ・未解決の観察を摩擦カードとして抽出
     - pain-log がない場合はスキップ（ユーザーにその場で聞く）

1-3. 刺激カードの収集
     - Glob: ideas/ 配下の daily-trend, ios-daily から直近1週間
     - Glob: knowledge/ 配下の TIL から直近1ヶ月
     - 面白い構造・パターン・トレンドを刺激カードとして抽出

1-4. カードの抽象化
     - 具体的すぎるカードは card-abstractor エージェントで抽象化
     - 2層構造（concrete + abstract）を維持

1-5. DEP-1: 過去出力チェック（オプション）
     - ideas/idea-mix/ に過去出力がある場合:
       diversity-context-loader を実行
         output_dir: ideas/idea-mix/
         lookback_count: 4
         extract_field: "## 生成概念" セクション
     - 初回実行の場合はスキップ
```

### Phase 2: カード選択

```
2-1. DEP-2: ローテーション設定
     rotation-engine をカスタムプールで実行:
       perspective: [分解的, 類推的, 反転的, 制約的, 再定義的, 時間軸的, スケール的]
       → 今週の Lens 構成を決定

2-2. 各デッキから1-2枚を選択
     - 能力カード: 1-2枚（ローテーション or ランダム）
     - 摩擦カード: 1-2枚（最近のpain-logから or ユーザー入力）
     - 刺激カード: 1枚（直近のトレンド/TILから）

2-3. ユーザーに選択結果を提示
     ---
     今回のカード:

     🃏 能力: {abstract表現}
        ({concrete表現})

     🔥 摩擦: {摩擦の記述}

     ⚡ 刺激: {刺激の記述}

     このカードで進めますか？ 変更したい場合は指示してください。
     ---

2-4. ユーザーの変更要望があれば反映
     - カードの差し替え
     - 追加のカード指定
     - テーマの絞り込み
```

### Phase 3: 認知的隔離実行（DEP-8）

```
3-1. Lens 構成の決定
     Phase 2 のローテーション結果に基づき、4つの Lens を構成:

     デフォルト構成:
       lens-A: 摩擦カードのみ + deconstruct（分解思考）
       lens-B: 刺激カードのみ + analogize（類推思考）
       lens-C: 能力カードのみ + invert（反転思考）
       lens-D: 能力+摩擦カード + constrain（制約思考）

     ローテーションにより thinking_frame は変動:
       例: 今週は [deconstruct, scale, reframe, temporal] になることもある

3-2. cognitive-isolation シーケンスを実行
     - 4つの Lens を並列起動（Agent tool × 4）
     - 各 Lens は部分情報のみ受け取る
     - output_format: "conclusion_only"（推論過程は破棄）

3-3. 結論を収集
     - 4つの結論（各200字以内）を取得
```

### Phase 4: 衝突合成（DEP-9）

```
4-1. 合成モードの決定
     ISO 週番号 % 3:
       0 → emergent（矛盾からの創発）
       1 → bridge（隠れた共通構造）
       2 → invert（暗黙の前提の反転）

4-2. collision-synthesis シーケンスを実行
     - 4つの結論を入力
     - synthesis_mode: 4-1 で決定したモード
     - concept_count: 3-5
     - novelty_constraint: Phase 1 で生成した制約（あれば）

4-3. 生成された概念をユーザーに提示
     ---
     ## 衝突から生まれた概念

     ### 1. {概念名}
     {説明}
     💥 衝突元: {どの結論の衝突から生まれたか}
     🆕 新規性: {なぜ新しいか}

     ### 2. {概念名}
     ...

     ### 3. {概念名}
     ...

     どれが面白いですか？ 深掘りしたい番号を教えてください。
     （複数可。「全部面白くない」も OK — カードを変えて再挑戦します）
     ---
```

### Phase 5: 対話的深掘り

```
5-1. ユーザーの選択を受け取る
     - 番号指定: 選択された概念を深掘り対象にする
     - 「全部面白くない」: Phase 2 に戻り、カードを変更して再実行
     - 「もっと出して」: Phase 4 を別の synthesis_mode で再実行

5-2. deepener エージェントの思考法で深掘り
     以下の順序で対話を進める:

     Step A: 存在チェック
       「これって既に〜として存在していませんか？」
       → ユーザーとの対話で差別化ポイントを明確化

     Step B: Devil's Advocate
       「もしこの概念が機能しないとしたら、最大の理由は？」
       → 3つの反論を提示し、ユーザーの見解を求める

     Step C: 本質の蒸留
       「この概念の核心を一言で言うと？」
       → 対話を通じて不要な要素を削ぎ落とす

     Step D: 最小実験の設計
       「明日からこれを検証するとしたら？」
       → 1日でできる検証方法を一緒に考える

5-3. 深掘り結果を構造化
```

### Phase 6: 記録

```
6-1. ファイル生成
     Write: ideas/idea-mix/YYYY-MM-DD-{slug}.md
     テンプレート: plugins/idea-mix/templates/output.md に従う

6-2. フロントマター:
     ---
     title: "{概念名}"
     created: YYYY-MM-DD
     tags: [idea-mix]
     status: raw
     collision:
       cards:
         capability: "{使用した能力カード}"
         friction: "{使用した摩擦カード}"
         stimulus: "{使用した刺激カード}"
       lens_frames: [{使用したフレーム一覧}]
       synthesis_mode: "{使用した合成モード}"
     novelty_score: {N}
     ---

6-3. Git コミット
     種別: ideas
     メッセージ: "ideas: idea-mix — {概念名}"
```

## PROOF

### Verification

- [ ] 4つの Lens が物理的に別コンテキストで実行されたか
- [ ] 各 Lens に部分情報のみ渡されたか
- [ ] 合成者は推論過程を見ていないか
- [ ] 生成された概念に novelty_score が付与されたか
- [ ] ユーザーとの対話で概念が研ぎ澄まされたか

### Done Criteria

- [ ] ideas/idea-mix/YYYY-MM-DD-{slug}.md が生成された
- [ ] 概念に「なぜ今まで存在しなかったか」の説明がある
- [ ] 最小実験が定義されている

### Do / Don't

**Do**:
- ✅ 各 Lens に異なる部分情報を渡す（情報遮断を徹底）
- ✅ ユーザーのカード変更要望を柔軟に受け入れる
- ✅ 「面白くない」結果が出たらカードを変えて再挑戦する
- ✅ 対話は1問ずつ、ペースを合わせる
- ✅ 生成概念の新規性を厳しくチェックする

**Don't**:
- ❌ ネット検索で既存アイデアを集めて提示する
- ❌ 全情報を1つのプロンプトに入れて考えさせる
- ❌ 推論過程を合成者に見せる
- ❌ 「面白そうですね」で深掘りを省略する
- ❌ 無理に結論を出す（面白くなければ再挑戦）
