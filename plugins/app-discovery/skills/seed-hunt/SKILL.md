---
title: "seed-hunt"
name: seed-hunt
description: "ideate platform — ネットの海からアプリアイデアの種を収集・スコアリングするスキル"
created: 2026-03-18
version: 2.0.0
tools: [Read, Write, Bash, WebSearch, WebFetch, Glob]
tags: [skill, app-discovery, ideation, diversity]
status: active
type: skill
trigger_keywords: ["seed hunt", "シードハント", "アイデア収集", "アプリネタ", "ペインポイント"]
output_dir: ideas/app-seeds/
diversity_deps: [DEP-1, DEP-2, DEP-6, DEP-7, DEP-4]
---

# seed-hunt

ネットの海（Reddit・Grok/X・はてブ・Yahoo知恵袋）からアプリアイデアの「種」を収集し、
6 軸スコアリング（P/F/M/G/Fit/**N**）で評価。Critic による多角的批判を経て
Featured 3 件 + Most Novel 1 件を毎回選出する。

ideate プラットフォームの多様性エンジニアリング（DEP-1/2/4/6/7）をフル実装した
週次ループスキル。

## CONTEXT

### 適用範囲

- **いつ**: 週次（月曜 or 日曜夜推奨）
- **用途**: アプリ・サービスのアイデア発見。`seed-validate` で検証するシードを供給
- **前提**: `pain-log` で蓄積した一次観察も参照するとシグナル精度が上がる

### 用語

| 用語 | 定義 |
|------|------|
| シード（Seed） | まだ検証されていないアイデアの候補。`seed-validate` が検証する |
| P スコア | Pain（痛みの深さ）: 1-5 |
| F スコア | Frequency（頻度）: 1-5 |
| M スコア | Market（市場規模）: 1-5 |
| G スコア | Gap（既存解との差）: 1-5 |
| Fit スコア | 個人開発適性: 1-5 |
| N スコア | Novelty（過去出力との新規性）: 1-5 |
| Featured | 今回の最優秀シード（Top 3 + Most Novel 1 件） |
| Novelty 制約 | DEP-1 の出力。過去 4 回分の Featured から生成した重複回避指示 |
| ローテーション設定 | DEP-2 の出力。今週の視点・クエリフレーム・追加ソース・アナロジードメイン |
| セレンディピティ枠 | DEP-6。通常 4 ソース収集後に analogy-finder で強制生成する 1 件 |

### 前提条件

- seed-hunt の出力ディレクトリが存在すること（デフォルト: `ideas/app-seeds/`。初回は空でよい）
- `XAI_API_KEY` 環境変数があれば Grok 検索が有効になる（optional）
- pain-log の出力ディレクトリがあれば Step 1 で参照する（デフォルト: `ideas/pain-log/`。optional）

---

## ROLE

| 属性 | 内容 |
|------|------|
| **専門性** | ペインポイント探索、アイデア評価、多様性エンジニアリング |
| **権限** | seed-hunt 出力ディレクトリへの書き込み（デフォルト: `ideas/app-seeds/`）、Web 検索・フェッチ |
| **責任** | 毎回質的に異なるシードを収集・評価し、Featured に価値あるシードを選出する |
| **禁止事項** | 本スキルの管轄ディレクトリ以外への書き込み |

---

## INTENT

### Goal

ネット上のペインポイントシグナルを 4 ソース（+ セレンディピティ 1 件）から収集し、
6 軸スコアリングと 3 フレーム Critic 批判を通じて、個人開発に適したアプリシードを
毎週新鮮に発見する。

### Value

- **V-1**: 多様性エンジニアリング（DEP-1/2/4/6/7）により、毎回異なる価値ある洞察が得られる
- **V-2**: 6 軸スコアリング（/30）で主観をできる限り排除した評価を実現
- **V-3**: Critic 批判により、スコアが高くても弱点を持つシードを事前に検出できる
- **V-4**: Most Novel 強制選出で、毎回少なくとも 1 件の新カテゴリシードを確保する

### Success Criteria

- [ ] 過去 4 回の Featured と意味的に重複しないシードが Featured の 80% 以上を占める
- [ ] 6 軸スコアリングが全シードに適用されている
- [ ] Critic 批判が Top 5 シードに対して 3 フレームで実行されている
- [ ] Most Novel（最高 N スコア）が Featured に強制選出されている
- [ ] 出力ファイルが出力ディレクトリの `YYYY-MM-DD.md`（デフォルト: `ideas/app-seeds/YYYY-MM-DD.md`）に保存されている

---

## STEPS

### Step 0: Memory-First（DEP-1 — Novelty 制約の生成）

`ideate/core/sequences/diversity-context-loader.md` を以下のパラメータで実行する:

```
output_dir:      seed-hunt の出力ディレクトリ（デフォルト: ideas/app-seeds/）
lookback_count:  4
extract_field:   "## Featured" セクションの ★ マーク付きシード（タイトル・カテゴリ・キーワード）
```

**処理**:

1. seed-hunt の出力ディレクトリ（デフォルト: `ideas/app-seeds/`）から直近 4 件のファイルを Glob で取得（日付降順）
2. 各ファイルの `## Featured` セクションから ★ シードのタイトル・カテゴリ・キーワードを抽出
3. `novelty_constraint`（重複回避プロンプト）を生成
4. `novelty_scoring_guide`（N スコア算出基準）を生成

初回実行（過去ファイル 0 件）の場合はこのステップをスキップして Step 0.5 へ進む。

**生成される Novelty 制約（抜粋イメージ）**:

```
## Novelty 制約（自動生成）
以下のシードは過去 4 回の Featured で既出です。意味的に重複する提案は避けてください。
- 献立管理 AI（カテゴリ: 生活・家事、keywords: [献立, レシピ, 料理]）
- タスク優先度 AI コーチ（カテゴリ: 生産性）
...
同カテゴリでもユーザーセグメントや解決アプローチが異なれば可。
```

---

### Step 0.5: Rotation Engine（DEP-2 — 今週の探索パラメータ決定）

`ideate/core/sequences/rotation-engine.md` を以下のカスタムプールで実行する:

```yaml
rotation_pools:
  perspective:
    - label: "経済的視点"
      prompt_hint: "コスト削減・収益損失・隠れたコストの観点でペインを評価せよ"
    - label: "感情的視点"
      prompt_hint: "フラストレーション・恥・孤独感・達成感の観点でペインを評価せよ"
    - label: "行動科学視点"
      prompt_hint: "習慣の中断・コミットメントのコスト・デフォルト効果の観点で評価せよ"
    - label: "技術者視点"
      prompt_hint: "自動化可能性・API 連携・既存ツールの欠落の観点で評価せよ"

  query_frame:
    - label: "問題探索"
      search_modifier: "困っている OR 不便 OR 面倒 OR ストレス OR 不満"
    - label: "解決策不満探索"
      search_modifier: "アプリ 不満 OR 使いにくい OR 欲しい機能 OR 代替"
    - label: "失敗・挫折分析"
      search_modifier: "やめた OR うまくいかない OR 諦めた OR デメリット OR 失敗"
    - label: "ニッチ需要探索"
      search_modifier: "需要 OR ニーズ OR マーケット OR ターゲット OR 欲しい"

  extra_source:
    - label: "ProductHunt"
      url_pattern: "site:producthunt.com"
    - label: "App Store レビュー"
      url_pattern: "site:apps.apple.com reviews"
    - label: "Quora"
      url_pattern: "site:quora.com"
    - label: "IndieHackers"
      url_pattern: "site:indiehackers.com"
    - label: "r/entrepreneur"
      url_pattern: "site:reddit.com/r/entrepreneur"
    - label: "Zenn/Qiita"
      url_pattern: "site:zenn.dev OR site:qiita.com"

  analogy_domain:
    - "製造業の予知保全システム"
    - "航空業界の安全チェックリスト運用"
    - "ゲームのオンボーディング・チュートリアル設計"
    - "病院のトリアージ・優先度管理"
    - "都市計画の渋滞緩和アルゴリズム"
    - "サプライチェーンの在庫最適化"
    - "ホテルのコンシェルジュサービス設計"
    - "農業の精密農業センシング"
    - "アダプティブラーニングの個別最適化"
    - "物流の最終マイル配送最適化"
    - "レストランの待ち行列マネジメント"
    - "保険のリスクアセスメントモデル"
    - "スポーツコーチングのパフォーマンス分析"
    - "軍のロジスティクス計画立案"
    - "図書館のレコメンデーションシステム"
    - "緊急サービスのディスパッチアルゴリズム"
    - "テーマパークの体験設計"
    - "金融の行動バイアス矯正プログラム"
    - "医療の服薬アドヒアランス支援"
    - "建築の空間認知・動線設計"
    - "カジノのエンゲージメント設計（倫理的転用）"
    - "軍隊のブリーフィング・デブリーフィング文化"
    - "消防の火災予防教育システム"
    - "科学実験のデータ記録・再現性管理"
    - "映画制作のプリプロダクション計画"
    - "コンタクトスポーツの怪我予防プロトコル"
    - "航空管制の状況認識維持"
    - "オリンピック選手のメンタルトレーニング"
    - "刑事司法の再犯防止プログラム"
    - "地下鉄のダイヤ最適化"
```

**決定論的計算**（手動実行の場合）:

```
perspective_index  = ISO週番号 % 4
query_frame_index  = 月 % 4
extra_source_index = (ISO週番号 + 3) % 6   ← perspective とずらす
analogy_domain_index = 日 % 30
```

**出力**: `rotation_context`（Step 1 以降のプロンプトに注入する設定テキスト）

フロントマターに記録: `rotation: {perspective.label}/{query_frame.label}`

---

### Step 1: pain-log シグナル参照（optional）

pain-log の出力ディレクトリ（デフォルト: `ideas/pain-log/`）の直近 2 週間分のファイルを Glob で取得し、
頻出カテゴリ・キーワードを抽出してシード収集時の優先ヒントとする。

```
pain-log が存在する場合:
  - 直近 14 日のエントリを読む
  - 3 回以上登場するカテゴリ・キーワードを "pain_hints" として記録
  - Step 2 の収集プロンプトに pain_hints を追加（"特にこのジャンルに注目"）

pain-log が存在しない場合:
  - このステップをスキップ
```

---

### Step 2: 4ソース収集（DEP-2 の rotation_context を注入）

以下の 4 ソースから並列収集する。全プロンプトに Step 0 の `novelty_constraint` と
Step 0.5 の `rotation_context` を先頭に注入する。

#### Source A: Reddit（英語ペインポイント）

**検索クエリ**:

```bash
# 基本クエリ（rotation の query_frame.search_modifier を含む）
q1="app idea reddit {query_frame.search_modifier}"
q2="I wish there was an app that site:reddit.com"
q3="why is there no app for site:reddit.com/r/technology"
q4="frustrating that site:reddit.com/r/productivity"
q5="someone should build site:reddit.com"
```

**curl 例（Bash ツールで実行）**:

```bash
curl -s "https://www.reddit.com/search.json?q=I+wish+there+was+an+app&limit=25&sort=new" \
  -H "User-Agent: Mozilla/5.0" | jq '.data.children[].data | {title, selftext, score, subreddit}'
```

```bash
curl -s "https://www.reddit.com/r/productivity/search.json?q=app+frustrating&limit=25&sort=top&t=month" \
  -H "User-Agent: Mozilla/5.0" | jq '.data.children[].data | {title, selftext, score, subreddit}'
```

```bash
curl -s "https://www.reddit.com/r/androidapps/search.json?q=wish+existed&limit=25&sort=top&t=week" \
  -H "User-Agent: Mozilla/5.0" | jq '.data.children[].data | {title, selftext, score, subreddit}'
```

**抽出対象**: upvote 数 > 10 または コメント数 > 5 のスレッドのペイン記述

**日本語フォローアップ**（WebSearch）:

```
WebSearch: "アプリ 欲しい OR 不便 OR 困っている site:reddit.com"
WebSearch: "{pain_hints の上位カテゴリ} 不便 reddit"
```

#### Source B: X/Twitter（Grok API — `XAI_API_KEY` が存在する場合）

**Grok API 呼び出し**:

```bash
curl -s "https://api.x.ai/v1/chat/completions" \
  -H "Authorization: Bearer $XAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "grok-3-latest",
    "messages": [
      {
        "role": "user",
        "content": "Search X/Twitter for recent tweets in Japanese and English about:\n1. \"アプリ 欲しい OR 不便 OR 困っている\"\n2. \"I wish there was an app that\"\n3. \"why doesn'\''t an app exist for\"\n\nReturn the top 15 most upvoted/engaging tweets with their content, likes, and retweet count. Focus on genuine pain points and unmet needs."
      }
    ],
    "search_parameters": {
      "mode": "on"
    }
  }' | jq '.choices[0].message.content'
```

**XAI_API_KEY が存在しない場合**:

```
WebSearch を使用してフォールバック:
WebSearch: "site:twitter.com アプリ 欲しい OR 不便"
WebSearch: "site:twitter.com I wish there was an app that"
```

#### Source C: はてなブックマーク（日本語テック・生活ニーズ）

**RSS フェッチ**:

```bash
# はてブ 新着エントリ（テクノロジー）
curl -s "https://b.hatena.ne.jp/hotentry/it.rss" | \
  python3 -c "
import sys, re
content = sys.stdin.read()
titles = re.findall(r'<title>(.*?)</title>', content)[2:]
descs = re.findall(r'<description>(.*?)</description>', content)[1:]
for t, d in zip(titles[:15], descs[:15]):
    print(f'TITLE: {t}\nDESC: {d}\n---')
"
```

```bash
# はてブ 人気エントリ（アプリ・ツール関連）
curl -s "https://b.hatena.ne.jp/search/tag?q=アプリ&users=5&sort=recent" \
  -H "User-Agent: Mozilla/5.0" | \
  grep -oP '(?<=class="entry-link">)[^<]+' | head -20
```

**追加 WebSearch**:

```
WebSearch: site:b.hatena.ne.jp アプリ 不便 OR 欲しい
WebSearch: site:b.hatena.ne.jp 困っている 解決策
WebSearch: はてブ 「{pain_hints の上位カテゴリ}」 アプリ 2026
```

#### Source D: Yahoo知恵袋（生活密着型ペイン）

**WebSearch 経由**:

```
WebSearch: site:detail.chiebukuro.yahoo.co.jp アプリ 欲しい OR 不便
WebSearch: site:detail.chiebukuro.yahoo.co.jp 困っている スマホ 解決
WebSearch: site:detail.chiebukuro.yahoo.co.jp "{pain_hints の上位カテゴリ}" 不便
```

**WebFetch でベストアンサー取得**:

```
search 結果 URL トップ 5 件を WebFetch して質問本文とベストアンサーを抽出
```

#### Source Extra: rotation.extra_source（ローテーション追加ソース）

`rotation_context` で指定された `extra_source` を WebSearch で探索する:

```
WebSearch: "{rotation.extra_source.url_pattern} app idea OR pain point {rotation.query_frame.search_modifier}"
```

例（ProductHunt の場合）:
```
WebSearch: "site:producthunt.com problems with OR I wish this app"
WebSearch: "site:producthunt.com 1-star review frustrating"
```

---

### Step 3: セレンディピティ枠（DEP-6 — analogy-finder）

通常の 4 ソース収集と並行して（または直後に）、
`ideate/core/agents/analogy-finder.md` を Agent tool で起動する。

```
Task(model: sonnet, agent: "ideate/core/agents/analogy-finder.md"):
  「
  ## アナロジー探索タスク

  元ドメイン: {rotation.analogy_domain}
  対象テーマ: アプリ・モバイルサービスのペインポイント解決
  制約: iOS アプリ または サービス、個人開発者が 3 ヶ月以内に MVP 開発可能

  ## Novelty 制約（注入）
  {novelty_constraint の内容}

  {analogy-finder.md の出力フォーマットに従い、推奨アイデア 1 件を返してください}
  」
```

**出力**: `serendipity_seed`（アイデア名・概要・借用元・差別化ポイント）

---

### Step 4: シグナル判定

各ソースから収集したテキストに対してシグナルワードフィルタを適用し、
ペインポイントの強さを 4 段階で判定する。

#### シグナルワード（強度別）

**L4 — 強い（High）**:
```
日本語: 毎日 / 毎週 / ずっと / もう限界 / 絶対に必要 / なんでないんだろう / 誰か作って
英語: every single day / desperately need / ruins my / can't believe there's no / someone please build
```

**L3 — 中（Medium）**:
```
日本語: よく / 結構 / いつも / 手動でやってる / めんどくさい / 不便
英語: often / keeps happening / annoying / manual workaround / frustrating / wish there was
```

**L2 — 弱（Weak）**:
```
日本語: たまに / ちょっと / 気になる / あれば便利
英語: sometimes / occasionally / would be nice / minor inconvenience
```

**L1 — ノイズ（Noise）**:
```
要望ではなく感想・ニュース・広告・スパム → 除外
```

**フィルタリングルール**: L3 以上のシグナルを持つアイデアのみ次ステップへ進める。

#### 週次カテゴリ（rotation で変わる視点を反映）

以下のカテゴリから今週のシードを分類する（rotation の `perspective` で優先カテゴリが変わる）:

| カテゴリ | 例 |
|---------|-----|
| 生活・家事 | 掃除, 料理, 買い物, 家計 |
| 健康・医療 | 服薬管理, 症状記録, 予防 |
| 仕事・生産性 | タスク管理, 集中, 会議 |
| お金・資産管理 | 家計, 投資, 節税 |
| 学習・自己成長 | 語学, 読書, スキルアップ |
| 人間関係・コミュニケーション | 家族, チーム, SNS |
| 趣味・エンタメ | ゲーム, 音楽, 旅行 |
| 子育て・教育 | 育児記録, 学習支援 |
| ペット | 健康管理, しつけ, 病院 |
| 移動・交通 | 通勤, 旅行計画, カーナビ |
| 環境・サステナビリティ | 節電, リサイクル, フードロス |
| シニア・介護 | 高齢者サポート, 見守り |

---

### Step 5: スコアリング（6 軸 /30）

L3 以上のシグナルを持つ全シードに対して以下の 6 軸でスコアリングする。
Step 0 の `novelty_scoring_guide` と `novelty_constraint` を参照して N スコアを算出する。

#### スコアリング軸定義

| 軸 | 項目 | 1 | 3 | 5 |
|----|------|---|---|---|
| **P** | Pain（痛みの深さ） | 軽微な不便 | 定期的なフラストレーション | 毎日の重大なブロッカー |
| **F** | Frequency（頻度） | 年に数回 | 週に数回 | 毎日 or 複数回/日 |
| **M** | Market（市場規模） | ニッチ（〜1万人） | ミドル（1万〜100万人） | マス（100万人以上） |
| **G** | Gap（既存解との差） | 良解が存在する | 既存解に不満点あり | 既存解が実質ない |
| **Fit** | 個人開発適性 | チーム・大資本が必須 | 個人でも可能だが困難 | 個人が 3 ヶ月以内で MVP |
| **N** | Novelty（過去出力との新規性） | 直近に同一あり | 類似あり | 過去 4 回に類似なし |

**N スコア詳細定義**（`novelty_scoring_guide` より）:

```
N = 5: past_items に類似なし（新カテゴリ または 全く新しいアプローチ）
N = 4: 同カテゴリだが切り口が明確に異なる
N = 3: 部分的に類似するが差別化ポイントがある
N = 2: 類似アイデアが past_items に存在する
N = 1: ほぼ同一のアイデアが past_items に存在する
```

**カットオフ**: Total ≥ 12/30 のシードのみ次ステップへ進める。

**`novelty_excluded` カウント**: N スコアが原因でカットオフを下回ったシード数を記録する。

---

### Step 6: Critic 批判（DEP-4 — Top 5 シードへの並列批判）

Total 上位 5 件のシードに対して、`ideate/core/sequences/parallel-critique.md` を実行する。

**批判フレーム（3 フレーム）**:

```yaml
critique_frames:
  - name: "技術的実現性"
    agent: ideate/core/agents/critic.md
    prompt: |
      以下の iOS アプリアイデアについて、技術的実現性の観点から批判してください。
      - 実装に必要なAPI・フレームワーク・インフラは現実的か
      - 個人開発者が Swift/SwiftUI で 3 ヶ月以内に MVP を作れるか
      - データ取得・プライバシー・リジェクトリスクなど技術的落とし穴は何か
      判定: PASS（実現可能）/ FAIL（実現困難）+ 根拠

  - name: "ユーザー心理"
    agent: ideate/core/agents/critic.md
    prompt: |
      以下の iOS アプリアイデアについて、ユーザー心理・行動の観点から批判してください。
      - ユーザーは本当にこの問題を「今すぐ」解決したいと思っているか
      - 既存のワークアラウンド（紙/Excel/習慣）を捨ててまでアプリを使うか
      - 継続利用・習慣化のモチベーションは何か、逆に離脱リスクは
      判定: PASS（使われる）/ FAIL（使われない）+ 根拠

  - name: "ビジネス成立性"
    agent: ideate/core/agents/critic.md
    prompt: |
      以下の iOS アプリアイデアについて、ビジネス成立性の観点から批判してください。
      - 収益化の道筋はあるか（サブスク / 買い切り / フリーミアム / 広告）
      - 市場規模は個人開発者が生存できる収益規模か（ARR 200-500万円以上）
      - 大手や類似アプリに対する持続的な優位性（ニッチ・速度・コミュニティ）はあるか
      判定: PASS（成立する）/ FAIL（成立しない）+ 根拠
```

**生存判定**: 3 フレーム中 2 つ以上 PASS → SURVIVED。それ以外 → ELIMINATED。

**survival_threshold**: 2（3 中 2）

---

### Step 7: Featured 選出

Critic 批判を経た SURVIVED シードから Featured を選出する。

**選出ロジック**:

```
1. SURVIVED シードを Total スコア降順でソート
2. Top 3 を Featured#1, #2, #3 として選出
3. SURVIVED シード全体（+ ELIMINATED シードも含む全シード）から
   最高 N スコアを持つシードを "Most Novel" として強制選出（既に Top 3 に含まれる場合は次点）
4. Featured の合計は最大 4 件（Top 3 + Most Novel）
```

**Most Novel 強制選出の意図**: DEP-7 の徹底実装。スコア上位に入らなくても
新しいカテゴリのシードを毎回 1 件確保し、パイプラインの多様性を維持する。

---

### Step 8: ファイル出力

出力ディレクトリの `YYYY-MM-DD.md`（デフォルト: `ideas/app-seeds/YYYY-MM-DD.md`）に以下のフォーマットで書き出す。

### 出力先の決定

本スキルは以下のデフォルトパスに出力する。ホストリポのディレクトリ構造が異なる場合は、
既存の構造に合わせて適切なディレクトリに配置すること。

| 出力 | デフォルトパス | ファイル命名 |
|------|-------------|------------|
| アイデアシート | `ideas/app-seeds/` | `YYYY-MM-DD.md` |

**配置ルール:**
- ホストリポに `ideas/` ディレクトリが存在すればその配下に配置
- 存在しない場合はリポルートの構造を確認し、類似のディレクトリ（`data/`, `output/`, `docs/` 等）に配置
- どちらもなければデフォルトパスでディレクトリを作成
- 過去の出力ファイルが既に別のパスに存在する場合はそのパスに従う

```markdown
---
title: "App Seeds — YYYY-MM-DD"
created: YYYY-MM-DD
tags: [app-seeds, seed-hunt]
status: active
week: YYYY-WNN
rotation: {perspective.label}/{query_frame.label}
sources: [reddit, grok, hatena, chiebukuro, {extra_source.label}]
total_collected: {件数}
after_signal_filter: {件数}
after_score_filter: {件数}
critic_survived: {件数}
novelty_excluded: {件数}
---

# App Seeds — YYYY-MM-DD

> **今週のローテーション**: {perspective.label} × {query_frame.label}
> **アナロジードメイン（セレンディピティ）**: {analogy_domain}

## Featured

### ★ 1. {シード名}
**カテゴリ**: {カテゴリ}
**ソース**: {ソース名 + URL}
**ペイン**: {ペイン記述 1-2 文}
**アイデア概要**: {解決アプローチ 2-3 文}
**スコア**: P:{n} F:{n} M:{n} G:{n} Fit:{n} N:{n} **Total: {合計}/30**
**Critic**: 技術:PASS / 心理:PASS / ビジネス:PASS（SURVIVED）
**改善ポイント**: {FAIL フレームの指摘があれば}

### ★ 2. {シード名}
（同形式）

### ★ 3. {シード名}
（同形式）

### ★ Most Novel — {シード名}
> ※ N スコア最高（N={n}）。Top 3 外だが新規性により強制選出。
**カテゴリ**: {カテゴリ}（過去 4 回未登場）
**ソース**: {ソース名 + URL}
**ペイン**: {ペイン記述}
**アイデア概要**: {解決アプローチ}
**スコア**: P:{n} F:{n} M:{n} G:{n} Fit:{n} N:{n} **Total: {合計}/30**
**Critic**: 技術:{判定} / 心理:{判定} / ビジネス:{判定}（SURVIVED / ELIMINATED）
**選出理由**: {なぜ新規性が高いか}

---

## Critic Results（Top 5）

| # | シード | 技術 | 心理 | ビジネス | 判定 |
|---|--------|------|------|---------|------|
| 1 | {タイトル} | PASS | PASS | FAIL | SURVIVED (2/3) |
| 2 | {タイトル} | PASS | PASS | PASS | SURVIVED (3/3) |
| 3 | {タイトル} | FAIL | FAIL | PASS | ELIMINATED (1/3) |
| 4 | {タイトル} | PASS | FAIL | PASS | SURVIVED (2/3) |
| 5 | {タイトル} | FAIL | PASS | FAIL | ELIMINATED (1/3) |

---

## All Seeds

| カテゴリ | シード名 | ソース | P | F | M | G | Fit | N | Total | Critic |
|---------|---------|--------|---|---|---|---|-----|---|-------|--------|
| {cat} | {名前} | {src} | {n} | {n} | {n} | {n} | {n} | {n} | {合計}/30 | SURVIVED/ELIMINATED/未批判 |
...

（Total ≥ 12 の全シードを記載。降順ソート）

---

## Serendipity（アナロジー探索）

**元ドメイン**: {analogy_domain}
**構造的類似性**: {共通する問題構造}
**生成シード**: {analogy-finder の推奨アイデア名}
**概要**: {借用元 + 適用方法}
**スコア**: P:{n} F:{n} M:{n} G:{n} Fit:{n} N:{n} **Total: {合計}/30**

---

## 今週のシグナルサマリー

**収集シード数**: {件数}（Reddit:{n} / X:{n} / はてブ:{n} / 知恵袋:{n} / Extra:{n} / Serendipity:1）
**シグナルフィルター後**: {件数}（L3以上）
**スコアカットオフ後**: {件数}（12/30以上）
**Novelty 除外数**: {novelty_excluded}件
**今週の強いシグナルカテゴリ**: {上位3カテゴリ}
**来週への引き継ぎ**: {気になるが今回のスコープ外だった観察 1-2 件}

---

## 次アクション

- [ ] `seed-validate` で Featured を検証（優先: #{番号}）
- [ ] `pain-log` に今週の新発見を追記
```

---

## Authority（権限）

| 操作 | 許可 |
|------|------|
| seed-hunt 出力ディレクトリへの書き込み（デフォルト: `ideas/app-seeds/`） | ✅ |
| pain-log 出力ディレクトリの読み取り（デフォルト: `ideas/pain-log/`） | ✅ |
| Web 検索・フェッチ | ✅ |
| Reddit curl コマンド実行 | ✅ |
| Grok API 呼び出し（$XAI_API_KEY） | ✅ |
| 本スキルの管轄ディレクトリ以外への書き込み | ❌ |
| `seed-validate` の自動起動 | ❌（ユーザー判断） |

---

## Limit（制限）

| 項目 | 上限 |
|------|------|
| 1 回の実行で収集するシード数 | 最大 40 件（各ソース最大 10 件）+ Serendipity 1 件 |
| スコアリング対象（フィルター後） | 最大 25 件 |
| Critic 批判対象 | Top 5 件（固定） |
| Featured 選出数 | 最大 4 件（Top 3 + Most Novel 1） |
| Reddit curl タイムアウト | 10 秒 |
| Grok API タイムアウト | 30 秒 |
| 1 ファイルの最大行数 | 400 行 |

---

## Signal（シグナルワード詳細）

ペインポイントの信頼性を上げるために、以下のワードパターンを優先する。

### 最強シグナル（L4）

```
日本語:
  "毎日 * が大変"、"ずっと * に困ってる"、"なんで * のアプリがないんだろう"
  "誰か * を作って"、"もう * は限界"、"* のせいで毎回時間を無駄にしてる"

英語:
  "every single day I have to manually"、"I desperately need an app that"
  "ruins my [workflow|morning|sleep|life]"、"can't believe there's no app for this"
  "someone please build"、"I've been waiting years for"
```

### 強シグナル（L3）

```
日本語:
  "よく * になる"、"いつも * が面倒"、"手動でやってる * "、"* が不便すぎる"
  "アプリで解決できないか"、"代替手段がない"

英語:
  "I always have to"、"it keeps"、"annoying that there's no"
  "wish someone would build"、"manual workaround for"、"frustrated by"
```

### 弱シグナル（L2）—収集はするがスコアを低めに設定

```
日本語: "たまに * が気になる"、"あれば便利かも"、"* は改善できそう"
英語: "would be nice if"、"minor annoyance"、"sometimes I wish"
```

---

## 他スキルとの棲み分け

| スキル | 役割 | seed-hunt との関係 |
|--------|------|-------------------|
| `pain-log` | 自分自身の一次観察をマイクロログ記録 | seed-hunt のシード収集ヒントを供給 |
| `seed-hunt` | ネット上のペインポイントからシードを収集 | **本スキル** |
| `seed-validate` | Featured シードの市場証拠を調査・検証 | seed-hunt の Featured を入力として受け取る |
| `seed-shape` | 検証済みシードを機能仕様・PRD に成形 | seed-validate 通過後のシードを受け取る |
| `pipeline-pulse` | パイプライン全体の週次レビュー | seed-hunt の出力をパイプライン状態として集計 |

---

## PROOF

### Output（成果物）

- ファイル: 出力ディレクトリの `YYYY-MM-DD.md`（デフォルト: `ideas/app-seeds/YYYY-MM-DD.md`）
- フォーマット: 上記「Step 8: ファイル出力」の Markdown テンプレート
- フロントマター: `title`, `created`, `tags`, `status`, `week`, `rotation`, `sources`,
  `total_collected`, `after_signal_filter`, `after_score_filter`, `critic_survived`, `novelty_excluded`

### Verification（検証方法）

- [ ] フロントマターに `rotation` フィールドが存在する
- [ ] `novelty_excluded` が 0 以上の整数で記録されている
- [ ] Featured に ★ Most Novel が含まれている
- [ ] All Seeds テーブルに N 列が存在し、全シードに値が入っている
- [ ] Critic Results テーブルに Top 5 件が記載されている
- [ ] Serendipity セクションが存在し、analogy_domain が記録されている
- [ ] Total スコアが /30 表記になっている

### Done Criteria（完了条件）

- [ ] 出力ディレクトリの `YYYY-MM-DD.md`（デフォルト: `ideas/app-seeds/YYYY-MM-DD.md`）が保存されている
- [ ] Featured が 3 件以上（Most Novel 含む最大 4 件）選出されている
- [ ] 全ステップ（Step 0 〜 Step 8）が実行されている
- [ ] Novelty 制約が今回のスコアリングに反映されている（N スコアが単調でない）

### Do / Don't

**Do**:
- ✅ Step 0 の Novelty 制約を **全収集プロンプトに注入** する（冒頭に貼る）
- ✅ Step 0.5 のローテーション設定をフロントマターに記録する
- ✅ Most Novel を Top 3 に関わらず **必ず** 1 件 Featured に含める
- ✅ N スコアは `novelty_scoring_guide` に従い、過去アイデアと比較して算出する
- ✅ curl コマンドが失敗した場合は WebSearch でフォールバックする
- ✅ ソースごとに最大 10 件に制限してノイズを抑制する
- ✅ シグナル L2 以下のシードは収集するがスコアを低めに設定する

**Don't**:
- ❌ Step 0 をスキップして Novelty 制約なしで収集しない（過去ファイルが 0 件の場合のみ例外）
- ❌ Total スコアをカットオフなしで全件 Critic にかけない（Top 5 のみ）
- ❌ Most Novel の強制選出を省略しない
- ❌ N スコアを推測で書かず、必ず past_items と比較して算出する
- ❌ 本スキルの管轄ディレクトリ以外にファイルを書き込まない
- ❌ Grok API が失敗しても全体を中断しない（Source B をスキップして続行）
- ❌ スコアリングを「感覚」で行わず、軸定義に従って 1-5 の根拠を持って評価する
