# ideate

AI-enhanced ideation platform — diverse, structured idea discovery powered by Claude Code plugins.

## Overview

ideate は、Claude Code のスキルシステム上に構築された **AI 強化アイデア発見基盤** である。
LLM の構造的な出力収束問題に対抗する「多様性エンジニアリング」を core に内蔵し、
各ドメイン特化プラグインが一貫した多様性戦略のもとでアイデアを発見・検証・成形する。

## Architecture

```
ideate/
├── core/                    ← 多様性エンジニアリング共通基盤
│   ├── sequences/           ← 共通ワークフロー
│   ├── agents/              ← 汎用エージェント
│   └── rules/               ← 設計原則・ルール
└── plugins/
    └── app-discovery/       ← アプリ・サービスアイデア発見
        ├── skills/          ← 5スキル（pain-log → seed-hunt → seed-validate → seed-shape → pipeline-pulse）
        ├── agents/          ← ドメイン特化エージェント
        └── templates/       ← 出力テンプレート
```

## Core: Diversity Engineering

LLM は同じプロンプトに対して同じ結論に収束する傾向がある。
ideate の core はこの問題に対抗する 7 つの設計原則を実装する:

1. **Memory-First** — 過去出力を記憶し、重複を回避する
2. **Input Diversity** — 入力（クエリ・ソース・視点）をローテーションで多様化する
3. **Reasoning Strategy Diversity** — ペルソナではなく推論戦略を変える
4. **Critic Diversity** — 提案より批判を多様化する
5. **Diverge-Converge Separation** — 発散と収束を明示的に分離する
6. **Serendipity Budget** — 探索の 20-30% を意図的な非効率に割り当てる
7. **Novelty as First-Class Metric** — 新規性をスコアリングの正式な評価軸にする

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

## Installation

```bash
claude plugins add github:no-problem-dev/ideate
```

## Requirements

- Claude Code CLI
- `XAI_API_KEY` environment variable (optional, for X/Twitter search via Grok API)

## License

MIT
