---
title: "Diversity Context Loader"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Diversity Context Loader

DEP-1（Memory-First）の実装。繰り返し実行スキルの冒頭で呼び出し、
過去出力から Novelty 制約を生成してプロンプトに注入する。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `output_dir` | 過去出力が保存されているディレクトリ | `ideas/app-seeds/` |
| `lookback_count` | 参照する過去ファイル数 | `4`（直近 4 回分） |
| `extract_field` | 抽出対象のフィールド/セクション | `Featured` セクションの ★ アイデア |
| `similarity_threshold` | 重複と判定する類似度閾値 | `0.8` |

## Steps

### Step 1: 過去出力の取得

```
1. Glob で output_dir から直近 lookback_count 件のファイルを取得
   - ファイル名の日付降順でソート
   - ファイルが存在しない場合は空リストで続行（初回実行対応）
2. 各ファイルから extract_field で指定されたセクションを抽出
   - Featured セクション、スコア上位 N 件、タイトル一覧など
   - スキルごとに抽出ロジックは異なる（パラメータで指定）
```

### Step 2: エッセンス抽出

```
各過去出力から以下を抽出し、コンパクトなリストを生成:
- アイデア/コンテンツのタイトル（1 行要約）
- カテゴリ/テーマ
- 主要なキーワード（3-5 個）

出力形式:
  past_items:
    - title: "献立管理 AI アプリ"
      category: "生活・家事"
      keywords: [献立, レシピ, 料理, 時短]
      date: 2026-03-10
    - title: "タスク優先度 AI コーチ"
      category: "仕事・生産性"
      keywords: [タスク管理, 優先度, 集中]
      date: 2026-03-04
    ...
```

### Step 3: Novelty 制約の生成

```
past_items リストから以下のプロンプト制約を生成:

---
## Novelty 制約（自動生成）

以下のアイデア/コンテンツは過去 {lookback_count} 回の出力で既出です。
**これらと意味的に重複する出力は避け、新しい切り口を優先してください。**

### 既出アイデア一覧
{past_items を箇条書きで列挙}

### 重複回避の指針
- 同じカテゴリでも、異なるユーザーセグメント・異なる解決アプローチなら可
- タイトルが異なっても、解決する問題が同じなら重複とみなす
- 過去に低スコアだったカテゴリは積極的に探索してよい
---
```

### Step 4: Novelty スコア算出基準の注入

```
スコアリングフェーズで使用する Novelty 基準を生成:

N（Novelty）スコア算出基準:
  5 = past_items に類似なし（新カテゴリまたは全く新しいアプローチ）
  4 = 同カテゴリだが切り口が明確に異なる
  3 = 部分的に類似するが差別化ポイントがある
  2 = 類似アイデアが past_items に存在する
  1 = ほぼ同一のアイデアが past_items に存在する
```

## Output

| 出力 | 説明 |
|------|------|
| `novelty_constraint` | プロンプトに注入する Novelty 制約テキスト |
| `novelty_scoring_guide` | スコアリング時の N スコア算出基準 |
| `past_items` | 抽出された過去アイデアのリスト |
| `past_item_count` | 過去アイデアの総数（メトリクス用） |

## 使用例

```markdown
# スキル冒頭での呼び出し

## Step 0: Diversity Context Loading

diversity-context-loader を以下のパラメータで実行:
- output_dir: ideas/app-seeds/
- lookback_count: 4
- extract_field: "## Featured" セクションの ★ マーク付きアイデア

生成された novelty_constraint を以降の全ステップのプロンプトに含める。
生成された novelty_scoring_guide をスコアリングステップで使用する。
```
