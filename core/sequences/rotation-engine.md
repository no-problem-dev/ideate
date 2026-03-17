---
title: "Rotation Engine"
created: 2026-03-18
version: 1.0.0
parent: core
type: sequence
---

# Rotation Engine

DEP-2（Input Diversity）の実装。日付ベースで入力パラメータを決定論的に切り替え、
同じスキルの繰り返し実行でも異なる入力空間を探索させる。

## Input

| パラメータ | 説明 | 例 |
|-----------|------|-----|
| `rotation_pools` | 各軸のローテーションプール定義 | 下記参照 |
| `date` | 基準日（省略時は当日） | `2026-03-18` |

## Steps

### Step 1: 日付からローテーション値を算出

```
各ローテーション軸について、日付のハッシュ値からインデックスを決定:

- perspective (視点): ISO 週番号 % pool_size
- query_frame (クエリフレーム): 月 % pool_size
- extra_source (追加ソース): ISO 週番号 % pool_size（perspective とは別プール）
- analogy_domain (アナロジードメイン): 日 % pool_size
```

### Step 2: ローテーション値の決定

各スキルが定義するプールから、Step 1 のインデックスで値を選択。

**デフォルトプール定義**（スキルがオーバーライド可能）:

```yaml
rotation_pools:
  perspective:
    - label: "経済的視点"
      prompt_hint: "コスト・収益・市場規模の観点から分析せよ"
    - label: "感情的視点"
      prompt_hint: "ユーザーの感情・フラストレーション・喜びの観点から分析せよ"
    - label: "行動科学視点"
      prompt_hint: "行動バイアス・習慣形成・動機づけの観点から分析せよ"
    - label: "技術者視点"
      prompt_hint: "技術的実現性・自動化可能性・スケーラビリティの観点から分析せよ"

  query_frame:
    - label: "問題探索"
      search_modifier: "困っている OR 不便 OR ストレス"
    - label: "解決策探索"
      search_modifier: "解決 OR 代替案 OR ワークアラウンド"
    - label: "失敗分析"
      search_modifier: "失敗 OR うまくいかない OR デメリット OR やめた"
    - label: "アナロジー探索"
      search_modifier: "〜のような OR に似た OR から学ぶ"

  extra_source:
    - label: "ProductHunt"
      url_pattern: "site:producthunt.com"
    - label: "App Store レビュー"
      url_pattern: "site:apps.apple.com レビュー"
    - label: "Quora"
      url_pattern: "site:quora.com"
    - label: "LinkedIn"
      url_pattern: "site:linkedin.com/pulse"
    - label: "専門フォーラム"
      url_pattern: "forum OR community"
    - label: "学術論文"
      url_pattern: "site:arxiv.org OR site:scholar.google.com"

  analogy_domain:
    - "製造業のメンテナンス管理"
    - "航空業界の安全プロトコル"
    - "ゲームのオンボーディング設計"
    - "病院の患者管理フロー"
    - "都市計画の交通設計"
    - "軍の指揮通信系統"
    - "ホテルのホスピタリティ設計"
    - "農業の精密管理"
    - "教育のアダプティブラーニング"
    - "物流の最適化アルゴリズム"
```

### Step 3: コンテキスト注入テキストの生成

```markdown
---
## 今回のローテーション設定（自動生成）

- **分析視点**: {perspective.label}
  → {perspective.prompt_hint}
- **クエリフレーム**: {query_frame.label}
  → 検索時に "{query_frame.search_modifier}" を含める
- **追加ソース**: {extra_source.label}
  → 通常ソースに加え {extra_source.url_pattern} も探索する
- **アナロジードメイン**: {analogy_domain}
  → セレンディピティ枠で「{analogy_domain}」の手法を対象ドメインに適用する

これらの設定に従い、通常とは異なる角度から探索・分析を行ってください。
---
```

## Output

| 出力 | 説明 |
|------|------|
| `rotation_context` | プロンプトに注入するローテーション設定テキスト |
| `perspective` | 今回の分析視点 |
| `query_frame` | 今回のクエリフレーム |
| `extra_source` | 今回の追加ソース |
| `analogy_domain` | 今回のアナロジードメイン |

## カスタマイズ

各スキルは `rotation_pools` をオーバーライドして、ドメイン固有のプールを定義できる。

```markdown
# スキル内でのオーバーライド例

rotation-engine を以下のカスタムプールで実行:
  perspective:
    - "B2C消費者視点"
    - "B2B企業視点"
    - "プラットフォーム視点"
    - "規制当局視点"
```
