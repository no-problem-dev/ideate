---
name: pipeline-pulse
description: >
  週末のアイデアパイプライン専用レビュー。pain-log + seed-hunt を横断分析し、
  P.A.I.N.フレームワークで整理、習慣ヘルスとフェーズ進捗を追跡する。
  「pipeline pulse」「パイプラインパルス」「パルス」「パイプラインレビュー」「週次アイデア」などで自動適用。
user-invocable: true
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion
---

# Pipeline Pulse スキル

## Authority
あなたは Life Repository のアイデアパイプラインレビューアとして振る舞う。
pain-log（一次観察）と seed-hunt（二次情報）を横断分析し、
P.A.I.N.フレームワークでパターンを抽出、習慣ヘルスを可視化する。

### /weekly との棲み分け

| | /weekly | /pipeline-pulse |
|--|---------|-----------------|
| 対象 | 全活動（コミット・Issue・学び・トレンド）| アイデアパイプラインのみ |
| 問い | 今週何が繰り返されたか？ | どんな課題が見えてきたか？次に検証すべきは？ |
| フレームワーク | 3つの問い | P.A.I.N. + パイプラインファネル |
| 習慣追跡 | なし | pain-log streak + フェーズ進捗 |
| 保存先 | knowledge/weekly/ | ideas/pulse/（デフォルト） |

### 自律レベル
| 行動 | レベル | 備考 |
|------|--------|------|
| データ収集・分析 | Level 4（自律） | pain-log, app-seeds, validation, shaping を自動読み込み |
| クラスター分析 | Level 4（自律） | AI でテーマ別グルーピング |
| P.A.I.N. 適用 | Level 4（自律） | 上位クラスターに自動適用 |
| Novelty ヘルス算出 | Level 4（自律） | 過去 4 週のデータから Novelty トレンドを追跡 |
| プロモート提案 | Level 2（対話） | ユーザーに昇格を確認 |
| ファイル作成 | Level 4（自律） | テンプレートに従い自動生成 |
| git commit | Level 4（自律） | 自動コミット |

## Target
週1回のパイプラインレビュー実施。習慣ヘルスの可視化とフェーズ進捗の追跡。
Novelty ヘルスにより「既視感アイデア」の増加を検知して多様性を維持する。

## Limit
- 出力は本スキルの管轄ディレクトリに限定する（デフォルト: `ideas/pulse/`）
- 他スキルの出力ディレクトリは読み取りのみ（pain-log, seed-hunt, seed-validate, seed-shape の各出力先）
- 既存の各スキル出力ファイルを変更しない
- プロモート（/seed-validate への昇格）はユーザー確認が必須

### 出力先の決定

本スキルは以下のデフォルトパスに出力する。ホストリポのディレクトリ構造が異なる場合は、
既存の構造に合わせて適切なディレクトリに配置すること。

| 出力 | デフォルトパス | ファイル命名 |
|------|-------------|------------|
| パルスレポート | `ideas/pulse/` | `YYYY-Wxx.md` |

**配置ルール:**
- ホストリポに `ideas/` ディレクトリが存在すればその配下に配置
- 存在しない場合はリポルートの構造を確認し、類似のディレクトリ（`data/`, `output/`, `docs/` 等）に配置
- どちらもなければデフォルトパスでディレクトリを作成
- 過去の出力ファイルが既に別のパスに存在する場合はそのパスに従う

## Action

### 使い方
```
/pipeline-pulse
```

### 処理手順

#### Step 1: Memory-First ロード（DEP-1）

過去のパルスデータを読み込んで Novelty トレンドを把握する:

1. **過去 4 週のパルスファイルを読み込む**
   - pipeline-pulse の出力ディレクトリ（デフォルト: `ideas/pulse/`）から直近 4 件の `*.md` ファイルを Glob で取得
   - 各ファイルのフロントマターと Novelty Health セクションを読み込む
   - 各週の「新規性 N≥4 アイデアの割合」を抽出する

2. **Novelty トレンドを算出**
   ```
   novelty_trend = [week-4, week-3, week-2, week-1] の新規性割合の推移
   ```
   - 前回のパルスが存在しない場合はトレンド比較をスキップ

3. **直近のアラート状態を確認**
   - 前回 Novelty Health セクションに ALERT が記載されていた場合はフラグを立てる

#### Step 2: データ収集（Read）

- pain-log の出力ディレクトリ（デフォルト: `ideas/pain-log/`） — 今週分（月〜日）の全エントリを読み込み
- seed-hunt の出力ディレクトリ（デフォルト: `ideas/app-seeds/`） — 今週分の全シートを読み込み
- seed-validate の出力ディレクトリ（デフォルト: `ideas/app-validation/`） — 直近のバリデーションファイルを確認
- seed-shape の出力ディレクトリ（デフォルト: `ideas/app-shaping/`） — 直近のシェイピングファイルを確認

#### Step 3: クラスター分析

- pain-log エントリをテーマ別に AI でグルーピング
- 各クラスターにエントリ数とキーワードを付与

#### Step 4: P.A.I.N. フレームワーク適用（ローテーション付き）— DEP-2

**週次ローテーション視点:**

実行週の ISO 週番号（`date +%V`）を Bash で取得し、4 で割った余りで視点を決定する:

| 週番号 mod 4 | 分析視点 | 問いの焦点 |
|-------------|---------|-----------|
| 0 | 経済的視点 | コスト・時間・収益の損失はどこか？ |
| 1 | 感情的視点 | フラストレーション・不安・恥ずかしさはどこか？ |
| 2 | 行動的視点 | 非効率なワークアラウンドはどこか？ |
| 3 | 技術的視点 | 既存ツールの技術的限界はどこか？ |

上位 3 クラスターに対し、その週の視点で P.A.I.N. フレームワークを適用する:

| P (People) | A (Activity) | I (Insights) | N (Needs) |
|---|---|---|---|
| 誰が困っている？ | 何をしようとしている？ | なぜうまくいかない？（今週の視点で） | 本当に必要なものは？ |

#### Step 5: クロスリファレンス

- pain-log のテーマが app-seeds にも出現しているか確認
- 重複テーマは信号強度が高いとマーク

#### Step 6: パイプラインファネル表示

```
観察: {N}件 → マイニング: {N}件 → 検証: {N}件 → 設計: {N}件
  pain-log     seed-hunt      seed-validate   seed-shape
```

#### Step 7: Novelty ヘルスダッシュボード（DEP-1 + DEP-2 統合）

**今週の Novelty スコア計算:**

seed-hunt のアイデアに対して以下で Novelty スコアを付与する:
- **N≥4（新規）**: 既存の app-seeds で見たことがないアプローチ
- **N=2-3（類似）**: 過去のアイデアと似ているが差分がある
- **N=1（既視感）**: 過去に何度も出現している同一テーマ

**Novelty Health 計算:**
```
novelty_rate = (今週の N≥4 アイデア数) / (今週の総アイデア数) × 100
```

**Novelty Health セクション:**
```
## Novelty Health

| 週 | 新規率 | アイデア総数 | N≥4 数 |
|----|--------|------------|--------|
| {現在-3週} | {N}% | {N} | {N} |
| {現在-2週} | {N}% | {N} | {N} |
| {現在-1週} | {N}% | {N} | {N} |
| **今週** | **{N}%** | **{N}** | **{N}** |

トレンド: {上昇↑ / 横ばい→ / 下降↓}
```

**アラート条件:**
```
IF novelty_rate < 30%:
  ALERT: 「新規性が低下しています（{N}%）。今週は /seed-hunt に多様性ヒントを追加することを推奨します」
  推奨アクション: seed-hunt の diverge-converge を活用する

IF novelty_rate < 30% かつ 前回も < 30%:
  CRITICAL ALERT: 「2 週連続で新規性が低下しています。マンネリ打破が必要です」
```

#### Step 8: 習慣ヘルスダッシュボード

```
| Metric | This Week | Monthly | All-time |
|---|---|---|---|
| Pain-log entries | {n} | {n}/30 | {n} |
| Pain-log streak | {n} days | best: {n} | - |
| Seed-hunt runs | {n} | {n} | {n} |
| Validations | {n} | {n} | {n} |
```

#### Step 9: フェーズ判定

- 累計 pain-log エントリ数でフェーズを判定:
  - Phase 1 Observer (0-3mo): 累計100件以上の pain-log を目指す
  - Phase 2 Experimenter (3-6mo): 3+ 個の MVP 出荷
  - Phase 3 Launcher (6-12mo): 初の有料ユーザー
  - Phase 4 Portfolio (12mo+): プロダクトポートフォリオ
```
Phase: {phase_name} (Phase {N})
Progress: {current}/{target} {metric}
Next milestone: {target} {metric} → Phase {N+1} ({next_phase_name})
```

#### Step 10: 候補プロモート（対話）

- 有望なクラスター（エントリ数が多い、Softwareable=yes が多い）を提示
- 「このテーマを `/seed-validate` で深掘りしますか？」と AskUserQuestion で確認
- ユーザーが Yes → Next Actions に記録

#### Step 11: ファイル作成

- テンプレート: `templates/pulse.md`
- 保存先: 出力ディレクトリの `YYYY-Wxx.md`（デフォルト: `ideas/pulse/YYYY-Wxx.md`、ISO 週番号）
- 各セクションを分析結果で埋める

#### Step 12: コミット

```bash
git add {output_dir}/YYYY-Wxx.md
git commit -m "ideas: Pipeline Pulse YYYY-Wxx"
```

## Signal
### 完了条件
- [ ] 出力ディレクトリの `YYYY-Wxx.md`（デフォルト: `ideas/pulse/YYYY-Wxx.md`）が作成されている
- [ ] 過去 4 週のパルスファイルが読み込まれている（または初回である旨が記載）
- [ ] クラスター分析が実施されている
- [ ] P.A.I.N. フレームワークが今週の視点（ローテーション）で上位クラスターに適用されている
- [ ] 今週の P.A.I.N. 視点が記載されている（経済的/感情的/行動的/技術的）
- [ ] Novelty Health セクションに過去 4 週のトレンドが記載されている
- [ ] novelty_rate < 30% の場合にアラートが記載されている
- [ ] パイプラインファネルが表示されている
- [ ] 習慣ヘルスダッシュボードが表示されている
- [ ] フェーズ判定が表示されている
- [ ] git commit が完了している

### 異常シグナル
| シグナル | 条件 | アクション |
|---------|------|-----------|
| INFO | 今週の pain-log が 0 件 | 「今週の観察記録がありません」と表示、習慣リマインド |
| INFO | 前回パルスが見つからない | Novelty トレンド比較をスキップ |
| INFO | 過去 4 週のデータが揃わない | 取得できた週数分のみで Novelty トレンドを算出 |
| WARN | seed-hunt の出力ディレクトリ（デフォルト: `ideas/app-seeds/`）が空 | seed-hunt セクションをスキップ |
| WARN | Novelty Health < 30% | アラートを Novelty Health セクションに記載 |
| ALERT | Novelty Health < 30% が 2 週連続 | CRITICAL ALERT を記載、多様性ヒントを提案 |
| WARN | git commit が失敗 | エラーを表示して手動コミットを案内 |
