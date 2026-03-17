---
title: "Diverge-Converge Separation"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Diverge-Converge Separation

DEP-5（発散-収束分離）の実装。制約を外した発散フェーズで解空間を広げ、
収束フェーズで制約を戻して実用的な提案に絞り込む。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `topic` | 探索対象のテーマ/問題 | "料理の献立決めが面倒" |
| `constraints` | 収束時に適用する制約リスト | ["iOS アプリ", "個人開発", "3ヶ月以内"] |
| `diverge_count` | 発散フェーズで生成する案の数 | `15` |
| `converge_count` | 収束フェーズで絞り込む案の数 | `3` |
| `selection_bias` | 絞り込み時の優先基準 | `"most_unconventional"` |

## Steps

### Step 1: 発散フェーズ（Diverge）

```
プロンプト:
「以下の問題/テーマについて、{diverge_count} 個の解決アプローチを列挙してください。

 テーマ: {topic}

 **重要なルール**:
 - プラットフォーム、技術、ビジネスモデル、ターゲット層の制約を全て外してください
 - 現実性、実装コスト、既存競合は一切考慮不要です
 - ハードウェア、サービス、コミュニティ、制度変更、教育、何でも含めてよい
 - 「こんなのありえない」と思うものほど歓迎します
 - 他ドメインの手法の転用、SF 的な発想、既存の常識の逆転も可

 {diverge_count} 個を番号付きリストで出力してください。
 各項目は 1-2 文の簡潔な説明で。」
```

### Step 2: 選択フェーズ（Select）

```
プロンプト:
「上記 {diverge_count} 個のアプローチから、
 **最も非典型的・意外性のある** {converge_count} 個を選んでください。

 選択基準:
 - selection_bias = "most_unconventional" の場合:
   「既存アプリで見たことがないアプローチ」を優先
 - selection_bias = "highest_impact" の場合:
   「実現できた場合のインパクトが最大のもの」を優先
 - selection_bias = "cross_domain" の場合:
   「他ドメインからの転用アプローチ」を優先

 選んだ {converge_count} 個の番号と選択理由を述べてください。」
```

### Step 3: 収束フェーズ（Converge）

```
プロンプト:
「選択された {converge_count} 個のアプローチを、
 以下の制約のもとで実現可能な形に具体化してください。

 制約:
 {constraints を箇条書きで列挙}

 各アプローチについて:
 1. 元のアイデアの核心（何が新しいか）
 2. 制約内での実現方法
 3. 制約によって失われるもの / 残るもの
 4. MVP として最小限必要な機能（3 つ以内）

 制約に合わない場合は、アイデアの核心を保ちつつ制約に収まる代替案を提示してください。」
```

## Output

| 出力 | 説明 |
|------|------|
| `diverged_ideas` | 発散フェーズで生成された全案リスト |
| `selected_ideas` | 選択された案 + 選択理由 |
| `converged_proposals` | 制約内で具体化された最終提案 |

## 使用例

```markdown
# seed-hunt スキルでの使用例

## セレンディピティ枠（7 件中 1 件）

diverge-converge を以下で実行:
  topic: 今週のカテゴリ「生活・家事」のペインポイント全般
  constraints: ["iOS アプリ", "個人開発 3 ヶ月以内", "Swift/SwiftUI"]
  diverge_count: 15
  converge_count: 2
  selection_bias: "cross_domain"

→ 通常の収集プロセスでは出てこないアイデアを 2 件強制生成
```
