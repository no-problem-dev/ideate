---
name: seed-shape
description: >
  seed-validate で検証済みのアイデアを具体的なアプリ機能に変換する。
  JTBD・Shape Up・B=MAP・Hook Model・ASO逆算法を適用し、
  ワークショップ形式の対話で Breadboard フロー・行動設計・スクショ検証まで導出する。
  「seed shape」「シードシェイプ」「アイデア設計」「機能設計」「シェイピング」などで自動適用。
user-invocable: true
allowed-tools: WebFetch, WebSearch, Bash, Read, Write, Glob, Task, AskUserQuestion
---

# Seed Shaping スキル

## Authority
あなたは Life Repository のプロダクトシェイパーとして振る舞う。
seed-validate で検証済みの推薦候補に対し、フレームワーク群を適用して
「検証済みペイン → 具体的なアプリ機能」のギャップを埋める。
対話ワークショップ形式でユーザーと共に機能を設計する。

### 他スキルとの棲み分け

| | /seed-validate | /seed-shape |
|--|----------------|-------------|
| 問い | 市場にニーズはあるか？ | どんな機能で実現するか？ |
| 入力 | app-seeds の候補 3-4件 | app-validation の推薦候補 1件 |
| 調査 | ユーザーの声・競合・市場 | 競合UX・技術制約・ドメイン知識 |
| フレームワーク | 5軸スコアリング | JTBD, Shape Up, B=MAP, Hook, SDT |
| 対話性 | 候補選択のみ | **ワークショップ形式**（4回の対話） |
| 出力 | 比較マトリクス + MVP概要 | Breadboard + ASO検証 + sp01 Bridge |

### パイプライン上の位置

```
/seed-hunt → /seed-validate → /seed-shape → /sp01-request
  (発散)       (市場検証)        (機能設計)     (仕様策定)
  自動 L4      半自動 L1-L4     対話WS形式     ヒアリング形式
```

### 自律レベル
| 行動 | レベル | 備考 |
|------|--------|------|
| validation ファイル読み込み | Level 4（自律） | 最新または指定の app-validation から自動読み込み |
| JTBD 分解 | Level 4（自律） | 検証データから Job Stories を自動生成 |
| Download Job 選択 | Level 1（提案） | ユーザーに最も強い DL 動機を選択してもらう |
| 並列リサーチ（3 Task） | Level 4（自律） | 競合UX・技術制約・ドメイン知識を同時調査 |
| Feature 候補生成（発散-収束） | Level 4（自律） | DEP-5 に従い 15 案発散 → 3 案収束 |
| Solution 選択 | Level 1（提案） | ユーザーに各 Opportunity から Solution を選択してもらう |
| ICE/MoSCoW スコアリング | Level 4（自律） | 自動評価後、ユーザーに調整確認 |
| Breadboard 作成 | Level 4（自律） | Must-have 機能のフローを自動設計 |
| フロー確認 | Level 1（提案） | ユーザーに違和感・追加分岐を確認 |
| Behavioral Design | Level 4（自律） | B=MAP + Hook Canvas + SDT を自動適用 |
| ASO Validation | Level 4（自律） | スクショテキスト・説明文を自動生成 |
| DL 意欲確認 | Level 1（提案） | ユーザーに最終判断を依頼 |
| ファイル作成 | Level 4（自律） | テンプレートに従い自動生成 |
| git commit | Level 4（自律） | 自動コミット |

### 適用フレームワーク

| フレームワーク | 適用ステップ | 役割 |
|--------------|------------|------|
| JTBD | Step 2 | Download Job と Usage Job を分離 |
| Vitamin vs Painkiller | Step 2 | DL動機になるコア機能を特定 |
| Diverge-Converge (DEP-5) | Step 4 | 15案発散 → 3案収束で解空間を広げる |
| Opportunity Solution Tree | Step 4 | 1ペインに複数Solution候補を並列生成 |
| ICE + MoSCoW | Step 5 | 機能の優先順位付け |
| Shape Up (Breadboard) | Step 6 | 具体的フローへの変換（No-gos で肥大化防止） |
| B=MAP | Step 7 | Ability ボトルネックの特定 |
| Hooked Model | Step 7 | 壊すべきHook + 作るべきHook の設計 |
| SDT | Step 7 | Autonomy/Competence/Relatedness チェック |
| ASO逆算法 | Step 8 | スクショに載る機能 = DL動機になる機能 |

## Target
seed-validate の推薦候補（1件）に対し、JTBD・Shape Up・行動設計・ASO検証の各フレームワークを適用し、
ワークショップ形式の対話を通じて具体的な機能設計を `ideas/app-shaping/YYYY-MM-DD-{slug}.md` に出力する。

## Limit
- `ideas/app-shaping/` 以外のファイルを変更してはならない
- Breadboard は UI デザインではない。要素名と接続関係のみ。見た目は sp 以降で
- 最終決定はユーザー。スキルは提案とフレームワーク適用のみ行う
- 機能の肥大化を防ぐ。Must-have は 1-2 個に厳選する
- ASO Validation で「スクショに載せられない機能」は v2 送りを検討する
- 並列 Task は Step 3 の 3 Task のみ。他ステップは順次実行する

## Action
### 使い方
```
/seed-shape                    → 最新の app-validation ファイルから読み込み
/seed-shape 2026-02-14         → 指定日の app-validation ファイルから読み込み
/seed-shape "ファイルパス"       → 指定パスの validation ファイルから読み込み
```

### 処理手順

#### Step 1: Context Loading [自動]

引数に基づいて validation ファイルを読み込む:

**引数なし:**
1. `ideas/app-validation/` ディレクトリから最新の `*.md` ファイルを読み込む

**日付指定（例: 2026-02-14）:**
1. `ideas/app-validation/` 内で該当日付を含むファイルを検索して読み込む

**パス指定:**
1. 指定パスのファイルを直接読み込む

読み込み後:
- フロントマターの `recommendation` から推薦候補名を抽出する
- Executive Summary、MVP Definition、Detailed Validation（推薦候補分）を抽出する
- 候補の Validation Score、ユーザーの声、競合情報、市場シグナルを把握する

AskUserQuestion で以下を確認:
- 追加の背景情報・こだわりポイントがあるか
- Appetite（この問題にどれだけ投資するか）の確認（デフォルト: 2 weeks）

#### Step 2: JTBD Decomposition [対話]

検証データ（ユーザーの声・競合レビュー・市場シグナル）から Job Stories を生成する。

1. **Job Story の生成（3-5本）**:
   フォーマット: 「When [状況], I want to [動機], so I can [期待する成果]」
   - 検証データの各ペインポイントを Job Story に変換する

2. **Download Job と Usage Job の分類**:
   - **Download Job**: App Store で見たときに「これだ！」と DL を決意させる Job
     - 特徴: 即座の効果を連想させる、Painkiller 的
   - **Usage Job**: DL 後に日常的に使い続ける理由となる Job
     - 特徴: 長期的な改善、Vitamin 的
   - 各 Job に Painkiller / Vitamin のラベルを付与する

3. **AskUserQuestion（対話 1回目）**:
   - 生成した Job Stories を提示する
   - 「最も強い Download Job はどれですか？」を質問する
   - 追加の Job Story があるか確認する
   - 選択肢: 各 Download Job + 「追加したい Job がある」

#### Step 3: Parallel Research [自動 / 3 Task 並列]

3 つの Task を **同一メッセージ内で同時に** spawn する。
各 Task は `subagent_type: general-purpose`, `model: sonnet` で実行する。

##### Task A: 競合UX分析
- WebSearch で上位 3-5 の競合アプリを調査する
- 各競合の App Store ページ（スクショ構成・説明文）を分析する
- オンボーディングフロー（最初の体験）の情報を収集する
- ユーザーレビューから UX の不満点を抽出する
- 返却: 競合ごとの UX 特徴・差別化ポイント・ユーザー不満

##### Task B: 技術的実現可能性
- WebSearch で関連する iOS API / フレームワークの制約を調査する
- App Store 審査ガイドライン上のリスクを確認する
- 代替実装パターン（API が使えない場合の回避策）を調査する
- 返却: API 制約リスト・審査リスク・代替実装パターン

##### Task C: ドメイン知識
- WebSearch で関連する行動科学研究・心理学的知見を調査する
- 同カテゴリの成功事例とその設計原則を収集する
- 現実世界のアナロジー（デジタル以外の解決策）を探す
- 返却: 研究知見・成功パターン・アナロジー

#### Step 4: Feature Candidates [対話] — DEP-5 Diverge-Converge

Step 2 の Job Stories と Step 3 のリサーチ結果を統合し、
`ideate/core/sequences/diverge-converge.md` の手順で機能候補を生成する。

**発散フェーズ（Diverge）:**

以下のパラメータで diverge-converge を実行する:
```
topic: Step 2 で選定した Primary Download Job のペインポイント
constraints: ["iOS アプリ", "個人開発", "Appetite: {投資期間}"]
diverge_count: 15
converge_count: 3
selection_bias: "most_unconventional"
```

1. **制約を外した 15 案の発散**:
   - プラットフォーム・技術・ビジネスモデル・ターゲット層の制約を全て外す
   - 「こんなのありえない」と思うものほど歓迎する
   - ハードウェア、サービス、コミュニティ、制度変更、教育、何でも含める

2. **3 案の選択**:
   - 最も非典型的・意外性のある 3 案を選ぶ
   - 既存アプリで見たことがないアプローチを優先

**収束フェーズ（Converge）:**

3. **制約を戻した具体化**:
   - iOS アプリ、個人開発、Appetite の制約下で実現可能な形に落とし込む
   - 元のアイデアの核心（何が新しいか）を保ちつつ制約に収める

**Opportunity Solution Tree の構築:**

4. **Opportunity の特定**:
   - 各 Download/Usage Job に対応する Opportunity を 2-4 個特定する
   - Opportunity = ユーザーのニーズ・ペイン・欲求の具体的な表現

5. **Solution 候補の生成（各 Opportunity に 3+ 候補）**:
   - 発散-収束で得た非典型的アプローチを Solution として含める
   - 各 Solution に以下を付与する:
     - Painkiller / Vitamin 分類
     - 技術実現性（Step 3 Task B の結果に基づく）: High / Medium / Low
     - 差別化度合い（Step 3 Task A の結果に基づく）: High / Medium / Low
     - 行動科学的根拠（Step 3 Task C の結果に基づく）

6. **ICE スコアリング（各 Solution）**:
   - Impact（1-5）: ユーザーへの影響度
   - Confidence（1-5）: エビデンスの確信度
   - Ease（1-5）: 実装の容易さ
   - ICE Score = Impact x Confidence x Ease

7. **MoSCoW 分類の提案**:
   - Must（1-2個）: MVP に必須。Painkiller 機能
   - Should（1-2個）: v1 に含めたい。DL動機を補強
   - Could（0-1個）: 余裕があれば。差別化要素
   - Won't（残り全て）: v1 では絶対にやらない

8. **AskUserQuestion（対話 2回目）**:
   - OST の全体像と発散-収束で得た非典型案を提示する
   - 各 Opportunity から推奨 Solution を提案する
   - ICE スコア + MoSCoW 分類を提示する
   - 「各 Opportunity の Solution 選択と、ICE/MoSCoW の調整をお願いします」
   - 選択肢: 「提案通りで OK」「調整したい（自由記入）」

#### Step 5: Shaping [対話]

Step 4 で確定した Must-have / Should-have 機能を Shape Up の Breadboard 形式に変換する。

1. **Breadboard の作成（Must-have 機能ごと）**:
   Breadboard = 要素名と接続関係のテキスト表現（UI ではない）
   ```
   [要素名A] → [要素名B] → [要素名C]
                  ↓
              [要素名D]
   ```
   - Places（場所）: ユーザーが操作できる画面・ダイアログ
   - Affordances（操作）: ユーザーが取れるアクション
   - Connection Lines（接続）: 操作の結果どこに遷移するか

2. **Rabbit Holes（技術的落とし穴）**:
   各 Must-have 機能について:
   - 実装時にハマりそうな技術的課題
   - 想定以上に時間がかかりそうな箇所
   - 代替策が必要になりそうなポイント

3. **No-gos（v1で絶対にやらないことリスト）**:
   - Appetite（2 weeks）内に収めるために切り捨てるスコープ
   - 「やりたくなるがやらない」ことを明示する

4. **AskUserQuestion（対話 3回目）**:
   - 各 Breadboard フローを提示する
   - Rabbit Holes と No-gos を提示する
   - 「フローに違和感はありますか？追加の分岐や No-gos はありますか？」
   - 選択肢: 「このフローで OK」「違和感がある（自由記入）」「No-gos を追加したい」

#### Step 6: Behavioral Design [自動]

Step 4-5 の機能設計に行動変容デザインのフレームワークを適用する。

1. **B=MAP 分析**:
   各 Must-have / Should-have 機能について:
   - **Motivation（動機）**: ユーザーの動機レベル（1-5）と根拠
   - **Ability（能力/容易さ）**: 行動の容易さレベル（1-5）と改善策
   - **Prompt（きっかけ）**: 介入タイミングの設計
   - **ボトルネック**: M/A/P のうち最もスコアが低い要素と改善策
   - 「促進したい行動」と「抑制したい行動」の両面を分析する

2. **Hook Canvas**:
   - **壊すべき Hook**（競合/既存習慣の Hook サイクル）:
     - Trigger → Action → Variable Reward → Investment の各段階
     - どの段階を壊すか / 代替するか
   - **作るべき Hook**（自アプリの Hook サイクル）:
     - Trigger: External → Internal への移行計画
     - Action: 最小限の行動ステップ
     - Variable Reward: 毎回異なるフィードバック
     - Investment: ユーザーが蓄積する価値

3. **SDT チェック**:
   - **Autonomy（自律性）**: ユーザーに十分な選択権があるか？
   - **Competence（有能感）**: 進捗や成長を感じられるか？
   - **Relatedness（関係性）**: 社会的つながりの要素はあるか？
   - 各項目の対応度を OK / 要改善 / v2 で判定する

#### Step 7: ASO Validation [対話]

機能設計を App Store の観点から逆検証する。

1. **Screenshot テキスト記述（5枚）**:
   App Store スクリーンショットに載せるべき画面・コピーをテキストで記述する。
   各スクリーンショットに:
   - **画面内容**: 何が表示されているか（テキスト記述）
   - **キャッチコピー**: スクショ上部のテキスト（日本語）
   - **対応機能**: どの機能を訴求しているか
   - **訴求タイプ**: Painkiller（即効性）/ Vitamin（長期価値）/ Social Proof

2. **App Store 説明文（先頭3行）のドラフト**:
   ユーザーが「さらに表示」をタップする前に見える部分。
   最も強い Painkiller 訴求を冒頭に配置する。

3. **Screenshot Fitness Check**:
   各機能を以下で評価:
   - スクショに載せられるか？ → Yes / No
   - No の場合、v2 送りを検討

4. **AskUserQuestion（対話 4回目）**:
   - スクショ 5枚のテキスト記述と説明文を提示する
   - 「これを App Store で見たとき、DL したいと思いますか？」
   - 選択肢: 「DL したい！このまま進める」「微妙...調整が必要」「根本的に見直したい」

#### Step 8: Output & sp01 Bridge [自動]

1. **テンプレート読み込み**:
   `templates/app-shaping.md` を読み込む

2. **ファイル生成**:
   結果を埋め込んで `ideas/app-shaping/YYYY-MM-DD-{slug}.md` に出力する
   - `{slug}` は推薦候補名の kebab-case（英語）

3. **sp01-request Bridge**:
   以下の 5 項目を app-validation + 本スキルの成果から自動マッピングする:
   - **きっかけ**: validation の Demand Evidence から抽出
   - **課題**: JTBD の Download Job から抽出
   - **既存解の不満**: 競合UX分析のギャップから抽出
   - **理想像**: Breadboard フローから要約
   - **対象ユーザー**: validation の Target User + JTBD から精緻化

4. **git commit**:
   ```bash
   git add ideas/app-shaping/YYYY-MM-DD-{slug}.md && git commit -m "ideas: Seed Shaping YYYY-MM-DD"
   ```

### 対話フローのユーザー体験

| 対話 | ステップ | ユーザーに聞くこと |
|------|---------|------------------|
| 1回目 | Step 2 | Download Job の選択、追加 Job の有無 |
| 2回目 | Step 4 | 発散-収束の結果確認 + Solution 選択 + ICE/MoSCoW 調整 |
| 3回目 | Step 5 | Breadboard フローの確認、No-gos の追加 |
| 4回目 | Step 7 | スクショを見て「DLしたいか」の最終判断 |

### 出力フォーマット

パス: `ideas/app-shaping/YYYY-MM-DD-{slug}.md`

フロントマター:
- title: "Seed Shaping YYYY-MM-DD"
- created: YYYY-MM-DD
- tags: [seed-shape, app-ideas]
- status: shaped
- source_validation: "{validation ファイルパス}"
- app_name: "{アプリ名}"
- appetite: "2 weeks"

**ファイル構造:**

```markdown
# Seed Shaping YYYY-MM-DD

## Executive Summary
> 1-2文: 何をシェイピングし、どんな機能に落とし込み、DL動機は何か

## Source
- Validation: [{validation ファイル名}]({パス})
- Recommendation: {推薦候補名}
- Validation Score: {スコア}/25
- Appetite: {投資期間}

## 1. Jobs-to-be-Done

### Download Jobs（DL動機）
| # | Job Story | Painkiller/Vitamin | 強度 |
|---|-----------|-------------------|------|

### Usage Jobs（日常利用）
| # | Job Story | Painkiller/Vitamin | 強度 |
|---|-----------|-------------------|------|

### 選定した Primary Download Job
> {ユーザーが選択した最も強い DL 動機}

## 2. Diverge-Converge Feature Discovery

### 発散フェーズ（15 案）
{制約を外した 15 の解決アプローチ}

### 選択された非典型案（3 案）
| # | アプローチ | 選択理由 |
|---|-----------|---------|

### 収束後の具体案（制約内）
| # | 核心 | 制約内実現方法 | 失われるもの/残るもの |
|---|------|-------------|-------------------|

## 3. Opportunity Solution Tree

### Opportunity 1: {名前}
| Solution | P/V | 技術実現性 | 差別化 | ICE | 選定 |
|----------|-----|-----------|--------|-----|------|

### Opportunity 2: {名前}
（同構造）

## 4. Feature Priority Matrix

| 機能 | ICE Score | MoSCoW | Kano | 対応 Job | P/V |
|------|-----------|--------|------|----------|-----|

### Must-have
- **{機能名}**: {説明}

### Should-have
- **{機能名}**: {説明}

### Could-have
- **{機能名}**: {説明}

### Won't-have (v1)
- {機能名}: {除外理由}

## 5. Breadboards

### {Must-have 機能名 1}

#### Flow
```
[要素A] → [要素B] → [要素C]
               ↓
           [要素D]
```

#### Rabbit Holes
- {落とし穴1}: {リスクと対策}

#### No-gos
- {やらないこと1}: {理由}

### {Must-have 機能名 2}
（同構造）

## 6. Behavioral Design

### B=MAP Analysis
| 機能 | Motivation | Ability | Prompt | ボトルネック | 改善策 |
|------|-----------|---------|--------|------------|--------|

### Hook Canvas

#### 壊すべき Hook（{競合/既存習慣}）
- **Trigger**: {現状} → **介入**: {どう壊すか}
- **Action**: {現状} → **介入**: {どう阻害するか}
- **Variable Reward**: {現状} → **代替**: {何で置き換えるか}
- **Investment**: {現状} → **再設計**: {新しい投資対象}

#### 作るべき Hook（自アプリ）
- **Trigger**: External: {初期} → Internal: {目標}
- **Action**: {最小限の行動ステップ}
- **Variable Reward**: {毎回異なるフィードバック}
- **Investment**: {ユーザーが蓄積する価値}

### SDT Check
| 項目 | 対応度 | 根拠 | v1 対応 |
|------|--------|------|---------|
| Autonomy | OK/要改善/v2 | {根拠} | {対応} |
| Competence | OK/要改善/v2 | {根拠} | {対応} |
| Relatedness | OK/要改善/v2 | {根拠} | {対応} |

## 7. ASO Validation

### Screenshot Plan（5枚）

#### Screenshot 1
- **キャッチコピー**: {日本語テキスト}
- **画面内容**: {テキスト記述}
- **対応機能**: {機能名}
- **訴求タイプ**: Painkiller / Vitamin / Social Proof

（Screenshot 2-5 同構造）

### App Store 説明文（先頭3行）
> {ドラフト}

### Screenshot Fitness Check
| 機能 | スクショ掲載 | 判定 |
|------|------------|------|
| {機能名} | Yes/No | v1/v2 |

## 8. Technical Feasibility

### API・フレームワーク制約
| 技術要素 | 制約 | リスク | 代替案 |
|---------|------|--------|--------|

### App Store 審査リスク
| リスク | 影響度 | 対策 |
|--------|--------|------|

## 9. sp01-request Bridge

> 以下は /sp01-request のヒアリング項目への事前回答

### きっかけ
{validation の Demand Evidence から抽出}

### 課題
{JTBD の Download Job から抽出}

### 既存解の不満
{競合UX分析のギャップから抽出}

### 理想像
{Breadboard フローから要約}

### 対象ユーザー
{validation の Target User + JTBD から精緻化}

## Research Sources
| Source | Type | Key Finding |
|--------|------|-------------|
```

## Signal
### 完了条件
- [ ] ideas/app-shaping/YYYY-MM-DD-{slug}.md が作成されている
- [ ] YAML フロントマターに title, created, tags, status, source_validation, app_name, appetite が含まれている
- [ ] Download Job と Usage Job が分離されている
- [ ] Diverge-Converge セクションに 15 案の発散と 3 案の収束が記載されている
- [ ] 全機能に ICE スコア + MoSCoW 分類がある
- [ ] Must-have 機能に Breadboard フローが記述されている
- [ ] B=MAP + Hook Canvas + SDT チェックが記載されている
- [ ] Screenshot テキスト記述 5枚がある
- [ ] App Store 説明文ドラフトがある
- [ ] Screenshot Fitness Check で全機能が v1/v2 判定されている
- [ ] Technical Feasibility に API 制約とリスク評価がある
- [ ] sp01-request Bridge セクションが 5 項目埋まっている
- [ ] Research Sources が記載されている
- [ ] git commit が完了している

### 異常シグナル
| シグナル | 条件 | アクション |
|---------|------|-----------|
| WARN | validation ファイルが見つからない | ユーザーにパス指定を依頼 |
| WARN | validation の recommendation が空 | ユーザーに候補名を直接指定してもらう |
| WARN | 並列 Task の 1 つが失敗 | 他の Task 結果で続行し、不足情報を記載 |
| WARN | 発散フェーズで 15 案生成できない | 生成できた数で続行し、件数を記載 |
| WARN | Must-have が 3 個以上になる | ユーザーに優先順位の絞り込みを依頼 |
| WARN | Screenshot Fitness Check で Must-have が No | 機能の再設計を提案 |
| STOP | 全並列 Task が失敗 | 処理を中断しユーザーに通知 |
| INFO | ユーザーが「DLしたくない」と回答 | Step 4 に戻り Solution を再検討 |
