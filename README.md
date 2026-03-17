# ideate

AI-enhanced ideation platform — diverse, structured idea discovery powered by Claude Code plugins.

## Overview

ideate は、Claude Code のスキルシステム上に構築された **AI 強化アイデア発見基盤** である。
LLM の構造的な出力収束問題に対抗する「多様性エンジニアリング」を core に内蔵し、
各ドメイン特化プラグインが一貫した多様性戦略のもとでアイデアを発見・検証・成形する。

## Architecture

```
ideate/
├── core/                    ← 多様性エンジニアリング共通基盤（DEP-1〜9）
│   ├── sequences/           ← 共通ワークフロー（rotation, diverge-converge, cognitive-isolation, collision-synthesis）
│   ├── agents/              ← 汎用エージェント（lens, critic, analogy-finder, collision-synthesizer）
│   └── rules/               ← 設計原則・ルール
└── plugins/
    ├── app-discovery/       ← アプリ・サービスアイデア発見
    │   ├── skills/          ← 5スキル（pain-log → seed-hunt → seed-validate → seed-shape → pipeline-pulse）
    │   ├── agents/          ← ドメイン特化エージェント
    │   └── templates/       ← 出力テンプレート
    └── idea-mix/            ← 汎用アイデア発想（認知的隔離+衝突合成）
        ├── skills/          ← 3スキル（idea-mix, idea-mix-deck, idea-mix-review）
        ├── agents/          ← card-abstractor, deepener
        └── templates/       ← 出力テンプレート
```

## Core: Diversity Engineering

LLM は同じプロンプトに対して同じ結論に収束する傾向がある。
ideate の core はこの問題に対抗する 9 つの設計原則を実装する:

1. **Memory-First** — 過去出力を記憶し、重複を回避する
2. **Input Diversity** — 入力（クエリ・ソース・視点）をローテーションで多様化する
3. **Reasoning Strategy Diversity** — ペルソナではなく推論戦略を変える
4. **Critic Diversity** — 提案より批判を多様化する
5. **Diverge-Converge Separation** — 発散と収束を明示的に分離する
6. **Serendipity Budget** — 探索の 20-30% を意図的な非効率に割り当てる
7. **Novelty as First-Class Metric** — 新規性をスコアリングの正式な評価軸にする
8. **Cognitive Isolation** — 異なるサブエージェントに部分情報を渡し、推論コンテキストの汚染を防ぐ
9. **Collision Synthesis** — 隔離された結論を衝突させ、単一視点では到達不能な概念を創発する

## Plugins

### app-discovery

アプリ・サービスのアイデアをペインポイントから発見・検証・成形するパイプライン。

| Skill | Description |
|-------|-------------|
| `/pain-log` | 日常の不満・不便を 30 秒でマイクロログ記録 |
| `/seed-hunt` | ネットの海からアプリアイデアの種を収集 |
| `/seed-validate` | 市場証拠に基づくアイデア検証 |
| `/seed-shape` | 検証済みアイデアを具体的な機能仕様に成形 |
| `/pipeline-pulse` | パイプライン全体の週次レビュー |

```
pain-log (一次観察)
    ↓
seed-hunt (ネット収集) → seed-validate (市場検証) → seed-shape (機能設計)
    ↓                                                      ↓
pipeline-pulse (週次レビュー)                         → /sp01-request (仕様策定へ)
```

### idea-mix

能力・摩擦・刺激のカードを認知的隔離（DEP-8）と衝突合成（DEP-9）で組み合わせ、
まだ存在しない概念を生成する汎用アイデア発想プラグイン。

| Skill | Description |
|-------|-------------|
| `/idea-mix` | カードを衝突させて新概念を生成する（メインスキル） |
| `/idea-mix-deck` | 能力・摩擦・刺激カードの管理（追加・一覧・同期・整理） |
| `/idea-mix-review` | 過去の結果を振り返り、効果的なパターンを分析 |

```
能力カード ─┐
            ├─→ Lens × 4 (認知的隔離) ─→ 衝突合成 ─→ 新概念 ─→ 対話的深掘り ─→ 記録
摩擦カード ─┤      各Lensは部分情報のみ       Opusが結論を衝突      ユーザーと研磨
            │      他Lensの存在を知らない
刺激カード ─┘
```

## Installation

```bash
claude plugins add github:no-problem-dev/ideate
```

## Requirements

- Claude Code CLI
- `XAI_API_KEY` environment variable (optional, for X/Twitter search via Grok API)

## License

MIT
