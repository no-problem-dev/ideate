---
name: seed-validate
description: >
  seed-hunt で収集したアイデア候補を深掘り検証する。
  ユーザーの声・競合分析・市場シグナルを証拠として収集し、
  5軸スコアリングで比較検証して MVP 定義まで導出する。
  「seed validate」「シード検証」「アイデア検証」「バリデーション」などで自動適用。
user-invocable: true
allowed-tools: WebFetch, WebSearch, Bash, Read, Write, Glob, Task, AskUserQuestion
---

# Seed Validation スキル

## Authority
あなたは Life Repository のプロダクトバリデーターとして振る舞う。
seed-hunt で収集されたアイデア候補に対し、ネット上のリアルなユーザーの声を証拠として収集し、
ビジネス実行性の観点から比較検証して推薦と MVP 定義を行う。

### 他スキルとの棲み分け

| | /seed-hunt | /seed-validate |
|--|------------|----------------|
| フェーズ | 発散（広く浅く） | 収束（狭く深く） |
| 入力 | 曜日カテゴリ（自動） | 3-4 件の具体的候補（ユーザー選択） |
| 検索方法 | カテゴリキーワードベース | 候補固有のペインポイントベース |
| Reddit | 固定 4 サブレディットの hot | グローバル検索 + 関連 sub（候補別） |
| Grok | 直近 1 日 | 直近 3 ヶ月（エビデンス量重視） |
| 追加ソース | なし | App Store 競合検索（WebSearch） |
| スコア軸 | Pain/Freq/MobileFit/Gap/Fit | DE/CG/MS/FE/MO（ビジネス実行性） |
| 出力 | 全件リスト | 比較マトリクス + MVP 定義 |

### 自律レベル
| 行動 | レベル | 備考 |
|------|--------|------|
| 候補ファイル読み込み | Level 4（自律） | 最新の app-seeds から自動読み込み |
| 候補選択 | Level 1（提案） | ユーザーに選択を依頼 |
| ソース巡回 | Level 4（自律） | 候補ごとに並列 Task で自動取得 |
| スコアリング | Level 4（自律） | エビデンス紐付きで 5 軸自動評価 |
| 推薦導出 | Level 1（提案） | 推薦のみ。最終決定はユーザー |
| MVP 定義 | Level 4（自律） | ユーザー声クラスタリングから導出 |
| ファイル作成 | Level 4（自律） | テンプレートに従い自動生成 |
| git commit | Level 4（自律） | 自動コミット |

## Target
seed-hunt で収集した候補（3-4 件）に対し、ユーザーの声・競合・市場シグナルを証拠として収集し、
Validation Score（5 軸）で比較検証した結果を `ideas/app-validation/YYYY-MM-DD-{slug}.md` に出力する。

## Limit
- `ideas/app-validation/` 以外のファイルを変更してはならない
- 1 ソースの失敗で全体を止めてはならない（他ソースで続行）
- 最終決定はユーザー。スキルは推薦のみ行う
- DE（Demand Evidence）が 2 以下の候補には「エビデンス不足」フラグを付ける
- `$XAI_API_KEY` が未設定の場合は X/Twitter ソースをスキップする
- MVP 定義は第一推薦候補のみに行う

## Action
### 使い方
```
/seed-validate              → 最新の app-seeds ファイルから候補選択
/seed-validate 2026-02-14   → 指定日の app-seeds ファイルから候補選択
/seed-validate "アイデアA, アイデアB, アイデアC"  → フリーテキストで候補指定
```

### Validation Score（5 軸）

seed-hunt の Seed Score（種の品質）と異なり、**ビジネス実行性**を評価する。

| 項目 | 略称 | 1 | 3 | 5 |
|------|------|---|---|---|
| Demand Evidence | DE | 声なし。仮説のみ | 3-5 件。中エンゲージ | 10+ 件。コミュニティ存在 |
| Competitive Gap | CG | 強い競合飽和 | 競合あるが明確なギャップ | 直接競合なし。未開拓 |
| Market Signal | MS | 関心低下 | 中程度の議論。安定 | トレンドトピック |
| Feasibility | FE | 新 API/HW 必要 | 標準 iOS パターン | 1-2 週 MVP 可能 |
| Monetization | MO | 支払い意欲なし | カテゴリに有料前例 | 強い支払い意欲 |

### 処理手順

#### Step 1: 候補の取得

引数に基づいて候補を取得する:

**引数なし or 日付指定の場合:**
1. `ideas/app-seeds/` ディレクトリから該当ファイルを読み込む
   - 引数なし → 最新の `*.md` ファイル
   - 日付指定（例: `2026-02-14`）→ `ideas/app-seeds/2026-02-14.md`
2. ファイル内の Featured（★）および全アイデアをリストアップする
3. AskUserQuestion で検証したい候補を 3-4 件選択してもらう

**フリーテキスト指定の場合:**
1. テキストをカンマ区切りでパースして候補リストとする

#### Step 2: 検索クエリ構築

各候補について以下を自動生成する:
- **コア KW（日本語）**: 候補のペインポイントを表す日本語キーワード 3-5 個
- **コア KW（英語）**: 同じ内容の英語キーワード 3-5 個
- **問題 KW**: 関連する困りごと・不満を表すキーワード
- **ソリューション KW**: 既存の解決策を探すキーワード

#### Step 3: 環境判定

Bash で以下のコマンドを **1 回で** 実行する:
```bash
echo "DATE=$(date +%Y-%m-%d)" && [ -n "$XAI_API_KEY" ] && echo "GROK=enabled" || echo "GROK=disabled"
```

#### Step 4: 候補ごとに並列 Task を spawn

**候補ごとに 1 Task**（3-4 Task 並列）を spawn する。
全 Task を **同一メッセージ内で同時に** spawn すること（並列実行の鍵）。
各 Task は `subagent_type: general-purpose`, `model: haiku` で実行する。

各 Task が以下の 3 つの調査を内部で順次実行する:

##### [A] User Voice Deep-Dive

**Reddit グローバル検索（3 ヶ月）:**
> **重要: WebFetch は reddit.com をブロックするため、必ず Bash ツールの curl を使うこと。**

```bash
# 候補固有のクエリでグローバル検索
curl -s -H "User-Agent: seed-validate/1.0" \
  "https://old.reddit.com/search.json?q={候補固有クエリ英語}&t=quarter&sort=relevance&limit=25" | \
  tr -d '\000-\037' | \
  jq -r '.data.children[] | "\(.data.title)|\(.data.ups)|\(.data.num_comments)|\(.data.selftext[:300])|\(.data.created_utc)|https://www.reddit.com\(.data.permalink)|\(.data.subreddit)"'
```

- グローバル検索（`/search.json`）で全サブレディットを横断
- `t=quarter` で直近 3 ヶ月の投稿を対象
- `selftext[:300]` で本文も取得（エビデンス抽出用）
- サブレディット名も取得してコミュニティの存在を確認

**Yahoo知恵袋検索:**
WebSearch で候補固有の問題 KW + シグナルワードで検索する。
```
site:detail.chiebukuro.yahoo.co.jp ({問題KW日本語}) ({コアKW日本語})
```

**X/Twitter Grok API（3 ヶ月範囲）:**
`$XAI_API_KEY` が設定されている場合のみ実行する。

```bash
if [ -z "$XAI_API_KEY" ]; then
  echo "SKIP: XAI_API_KEY not set"
  exit 0
fi

THREE_MONTHS_AGO=$(date -v-3m +%Y-%m-%d)
TODAY=$(date +%Y-%m-%d)

RESPONSE=$(curl -s -w "\n%{http_code}" -X POST "https://api.x.ai/v1/responses" \
  -H "Authorization: Bearer $XAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "grok-4-1-fast",
    "tools": [{"type": "x_search", "x_search": {"from_date": "'"$THREE_MONTHS_AGO"'", "to_date": "'"$TODAY"'"}}],
    "input": [{
      "role": "user",
      "content": "X投稿を検索して、「{候補名}」に関するユーザーの困りごと・不満・要望を見つけてください。検索キーワード: {候補固有KW} AND (困る OR 不便 OR 面倒 OR 欲しい OR アプリ OR wish OR frustrated OR need)。各件について: (1) ペインポイント要約（1行）、(2) 元投稿の抜粋、(3) エンゲージメント（いいね・RT数）を教えてください。英語・日本語両方の投稿を含めてください。"
    }]
  }')

HTTP_CODE=$(echo "$RESPONSE" | tail -1)
BODY=$(echo "$RESPONSE" | sed '$d')

if [ "$HTTP_CODE" != "200" ]; then
  echo "WARN: Grok API returned HTTP $HTTP_CODE"
  exit 0
fi

echo "$BODY" | jq -r '.output[] | select(.type == "message") | .content[] | select(.type == "output_text") | .text'
```

##### [B] Competitive Analysis

**App Store 競合検索:**
WebSearch で以下のクエリを実行する:
```
site:apps.apple.com {候補固有のソリューションKW英語} app
```

**競合レビュー・不満検索:**
WebSearch で以下のクエリを実行する:
```
"{競合アプリ名}" (review OR レビュー OR 不満 OR "doesn't work" OR "missing feature")
```

- 上位 3-5 件の競合アプリを特定する
- 各競合の主な機能・評価・不満点を要約する
- 現在の競合が満たしていないギャップを特定する

##### [C] Market Signals

**はてなブックマーク検索 RSS:**
> **重要: WebSearch の `site:b.hatena.ne.jp` は結果が返らないため使わない。はてブの検索 RSS を curl で取得すること。**

```bash
curl -s "https://b.hatena.ne.jp/search/text?q={候補固有KW URL エンコード済み}&mode=rss"
```

- `<item>` からタイトル・リンク・`<hatena:bookmarkcount>` を抽出
- ブックマーク数でトレンド度合いを判定

**ブログ・記事検索:**
WebSearch で以下のクエリを実行する:
```
{候補固有KW日本語} (解決 OR アプリ OR ツール OR サービス)
{候補固有KW英語} (solution OR app OR tool OR service)
```

- 関連記事の量と鮮度からマーケットの関心度を判定する

**Task の返却フォーマット:**
各 Task は以下の構造でテキスト結果を返す:
```
## Candidate: {候補名}

### User Voice
- [Reddit] {ペインポイント要約} (upvotes: N, comments: N) — URL
- [知恵袋] {質問要約} — URL
- [X/Twitter] {ペインポイント要約} (likes: N) — エビデンステキスト
（または SKIP: XAI_API_KEY not set）

### Competitive Landscape
- {アプリ名} (★{評価}) — {主な機能} — {ギャップ}
- ...

### Market Signals
- [はてブ] {タイトル} ({N} users) — URL
- [記事] {タイトル} — URL
- ...
```

#### Step 5: 結果統合 → 5 軸スコアリング

全 Task の結果を統合し、各候補に Validation Score を付与する。

**スコアリング基準（エビデンス紐付き）:**

| 項目 | 略称 | 1 | 3 | 5 |
|------|------|---|---|---|
| Demand Evidence | DE | 声なし。仮説のみ | 3-5 件。中エンゲージ | 10+ 件。コミュニティ存在 |
| Competitive Gap | CG | 強い競合飽和 | 競合あるが明確なギャップ | 直接競合なし。未開拓 |
| Market Signal | MS | 関心低下 | 中程度の議論。安定 | トレンドトピック |
| Feasibility | FE | 新 API/HW 必要 | 標準 iOS パターン | 1-2 週 MVP 可能 |
| Monetization | MO | 支払い意欲なし | カテゴリに有料前例 | 強い支払い意欲 |

各スコアにはエビデンス（具体的な URL やデータ）を紐付けること。

#### Step 5.5: DEP-4 Critic Diversity（並列批判）

スコアリング完了後、全候補に対して `ideate/core/sequences/parallel-critique.md` の手順で並列批判を実行する。

**批判フレーム（3種）:**
1. **技術的実現性**: 個人開発者が 3 ヶ月以内に MVP を作れるか
2. **ユーザー心理**: 既存の習慣を捨ててまで使うか
3. **ビジネス成立性**: 収益化の道筋と市場規模は十分か

**survival_threshold**: 3 中 2 通過で SURVIVED

批判結果を Comparison Matrix に `Critique` 列として追加する:
- SURVIVED (2/3): 生存。強みと改善ポイントを注釈
- SURVIVED (3/3): 完全生存。高信頼
- ELIMINATED (1/3 以下): 排除。理由を記録

#### Step 6: 比較マトリクス生成

| Dimension | A: {名前} | B: {名前} | C: {名前} |
|-----------|-----------|-----------|-----------|
| DE | N/5 | N/5 | N/5 |
| CG | N/5 | N/5 | N/5 |
| MS | N/5 | N/5 | N/5 |
| FE | N/5 | N/5 | N/5 |
| MO | N/5 | N/5 | N/5 |
| **Total** | **N/25** | **N/25** | **N/25** |
| **Critique** | SURVIVED/ELIMINATED | SURVIVED/ELIMINATED | SURVIVED/ELIMINATED |

#### Step 7: 推薦導出

- Total Score が最高の候補を第一推薦とする
- DE ≤ 2 の候補には「エビデンス不足」フラグを付ける
- Critique が ELIMINATED の候補には「批判フィルタ不通過」フラグを付ける
- 推薦理由を 2-3 文で記載する（エビデンス参照付き）
- 見送り候補には見送り理由を明記する

#### Step 8: MVP 定義（第一推薦のみ）

ユーザーの声をクラスタリングし、以下を導出する:

- **ターゲットユーザー**: ペルソナ定義（収集した声から導出）
- **Core Features（3-5 機能）**: 各機能にユーザー声の根拠 URL を紐付ける
- **NOT in MVP**: スコープ外とする機能
- **技術スタック想定**: 開発者プロファイルに基づく
- **想定開発期間**: 1-2 週 or 2-4 週 or 1 ヶ月超

#### Step 9: ファイル作成 → git commit

1. `templates/app-validation.md` を読み込む
2. 結果を埋め込んで `ideas/app-validation/YYYY-MM-DD-{slug}.md` に出力する
   - `{slug}` は第一推薦候補名の kebab-case（英語）
3. git commit する

```bash
git add ideas/app-validation/YYYY-MM-DD-{slug}.md && git commit -m "ideas: Seed Validation YYYY-MM-DD"
```

### 出力フォーマット

パス: `ideas/app-validation/YYYY-MM-DD-{slug}.md`

フロントマター:
- title: "Seed Validation YYYY-MM-DD"
- created: YYYY-MM-DD
- tags: [seed-validate, app-ideas]
- status: validated
- candidates: {候補数}
- recommendation: "{第一推薦候補名}"

**ファイル構造:**

```markdown
# Seed Validation YYYY-MM-DD

## Executive Summary
> 1-2 文: 何を検証し、何が分かり、何を推薦するか

## Candidates
| # | Candidate | Origin Score | Source |
|---|-----------|-------------|--------|
| A | {名前} | {元の Seed Score}/25 | app-seeds {日付} |
| B | {名前} | {元の Seed Score}/25 | app-seeds {日付} |
| C | {名前} | {元の Seed Score}/25 | app-seeds {日付} |

## Comparison Matrix
| Dimension | A: {名前} | B: {名前} | C: {名前} |
|-----------|-----------|-----------|-----------|
| Demand Evidence (DE) | N/5 | N/5 | N/5 |
| Competitive Gap (CG) | N/5 | N/5 | N/5 |
| Market Signal (MS) | N/5 | N/5 | N/5 |
| Feasibility (FE) | N/5 | N/5 | N/5 |
| Monetization (MO) | N/5 | N/5 | N/5 |
| **Total** | **N/25** | **N/25** | **N/25** |
| **Critique** | SURVIVED(N/3) | SURVIVED(N/3) | ELIMINATED(N/3) |

## Critique Results

### Critic Diversity（並列批判）
| # | Candidate | 技術 | 心理 | ビジネス | 判定 |
|---|-----------|------|------|---------|------|
| A | {名前} | PASS/FAIL | PASS/FAIL | PASS/FAIL | SURVIVED/ELIMINATED |

### Survived Candidates
{各 SURVIVED 候補の強み + 改善ポイント}

### Key Insights from Eliminated
{ELIMINATED 候補から学べる弱点パターン}

## Detailed Validation

### Candidate A: {名前}

#### Demand Evidence (DE: N/5)
- [Reddit] {ペインポイント要約} (upvotes: N) — [リンクテキスト](URL)
- [知恵袋] {質問要約} — [リンクテキスト](URL)
- [X/Twitter] {ペインポイント要約} (likes: N)
> 判定根拠: {なぜこのスコアか}

#### Competitive Landscape (CG: N/5)
- **{競合アプリ1}** (★{評価}): {主な機能} — ギャップ: {不足点}
- **{競合アプリ2}** (★{評価}): {主な機能} — ギャップ: {不足点}
> 判定根拠: {なぜこのスコアか}

#### Market Signals (MS: N/5)
- [はてブ] {タイトル} ({N} users) — [リンクテキスト](URL)
- [記事] {タイトル} — [リンクテキスト](URL)
> 判定根拠: {なぜこのスコアか}

#### Feasibility (FE: N/5)
- 必要技術: {技術要素}
- 推定開発期間: {期間}
> 判定根拠: {なぜこのスコアか}

#### Monetization (MO: N/5)
- 課金モデル案: {モデル}
- 同カテゴリの有料アプリ前例: {例}
> 判定根拠: {なぜこのスコアか}

（候補 B, C も同構造）

## Recommendation

### 第一推薦: {候補名}
{推薦理由 2-3 文。エビデンス参照付き}

### 次点: {候補名}
{理由}

### 見送り: {候補名}
{見送り理由}

## MVP Definition

> 以下は第一推薦「{候補名}」の MVP 定義

### Target User
{ペルソナ定義: 収集した声から導出した具体的なターゲット像}

### Core Features
1. **{機能名}**: {説明}
   - 根拠: [{ユーザー声の要約}](URL)
2. **{機能名}**: {説明}
   - 根拠: [{ユーザー声の要約}](URL)
3. **{機能名}**: {説明}
   - 根拠: [{ユーザー声の要約}](URL)
（3-5 機能）

### NOT in MVP
- {スコープ外機能1}: {理由}
- {スコープ外機能2}: {理由}

### Tech Stack
- {技術スタック}（開発者プロファイルに基づく）

### Estimated Timeline
- {想定期間}

## Next Steps
- [ ] `/seed-shape` で機能設計を開始
  - 候補名: {第一推薦候補名}
  - 検証ファイル: `ideas/app-validation/YYYY-MM-DD-{slug}.md`
  - MVP Core Features を seed-shape の初期要件として使用

## Data Sources & Methodology
| Source | Status | Queries | Results |
|--------|--------|---------|---------|
| Reddit (Global Search) | OK/WARN/SKIP | {クエリ数} | {結果件数} |
| Yahoo知恵袋 | OK/WARN/SKIP | {クエリ数} | {結果件数} |
| X/Twitter (Grok 3mo) | OK/WARN/SKIP | {クエリ数} | {結果件数} |
| App Store (WebSearch) | OK/WARN/SKIP | {クエリ数} | {結果件数} |
| はてなブックマーク | OK/WARN/SKIP | {クエリ数} | {結果件数} |
| ブログ・記事 | OK/WARN/SKIP | {クエリ数} | {結果件数} |
```

## Signal
### 完了条件
- [ ] ideas/app-validation/YYYY-MM-DD-{slug}.md が作成されている
- [ ] YAML フロントマターに title, created, tags, status, candidates, recommendation が含まれている
- [ ] 全候補に 5 軸スコア + エビデンス URL が紐付いている
- [ ] 比較マトリクスに Critique 列が含まれている
- [ ] Critique Results セクションに並列批判の結果が記載されている
- [ ] 第一推薦に MVP Definition（3-5 機能 + 根拠）が記載されている
- [ ] DE ≤ 2 の候補に「エビデンス不足」フラグが付いている
- [ ] Next Steps に /seed-shape への接続情報がある
- [ ] Data Sources & Methodology テーブルに各ソースの状況が記載されている
- [ ] git commit が完了している

### 異常シグナル
| シグナル | 条件 | アクション |
|---------|------|-----------|
| WARN | 1 つのソースの取得に失敗 | 他のソースで続行し、Data Sources テーブルに記録 |
| WARN | $XAI_API_KEY が未設定 | Grok API をスキップ。他ソースで続行 |
| WARN | Grok API 呼び出しがエラー | X/Twitter ソースをスキップ、他ソースで続行 |
| WARN | App Store 検索で競合が見つからない | CG スコアを「判断不能」とし理由を記載 |
| WARN | Critique の並列 Task が失敗 | 利用可能なフレームの結果で続行し、欠損を明記 |
| STOP | 全ソースの取得に失敗 | 処理を中断しユーザーに通知 |
| WARN | 候補数が 2 件未満 | 処理は続行するが比較の有効性について注記 |
| INFO | 全候補の DE ≤ 2 | エビデンス不足の旨を Executive Summary に記載 |
