---
title: "Idea Mix Deck"
created: 2026-03-18
version: 1.0.0
context: main
model: claude-sonnet-4
tools: [Read, Write, Edit, Glob, Grep]
tags: [skill, idea-mix, deck-management]
status: active
type: skill
user_invocable: true
trigger: ["/idea-mix-deck", "カード管理", "能力カード", "デッキ管理"]
---

# Idea Mix Deck

## CONTEXT

### 適用範囲

idea-mix プラグインのカードデッキを管理するスキル。
能力・摩擦・刺激の3種類のカードの追加・一覧・抽象化・整理を行う。

### 前提条件

- `ideas/idea-mix/deck/` ディレクトリが存在すること（なければ作成）

## ROLE

| 属性 | 内容 |
|------|------|
| **専門性** | カードの分類、抽象化レベルの管理、デッキバランスの維持 |
| **権限** | カードファイルの読み書き |
| **責任** | idea-mix に使える高品質なカードデッキの維持 |

## INTENT

### Goal

idea-mix の入力となるカードデッキを管理し、常に使えるカードが揃っている状態を維持する。

## STEPS

### サブコマンド

#### `add` — カードの追加

```
使い方: /idea-mix-deck add "記述"

1. 記述からカード種別を判定
   - 「〜ができる」「〜を作れる」→ capability
   - 「〜が不便」「〜がストレス」→ friction
   - 「〜が面白い」「〜というパターン」→ stimulus

2. card-abstractor エージェントで抽象化
   - concrete: 元の記述
   - abstract: 抽象化された記述

3. 該当するデッキファイルに追記
   - ideas/idea-mix/deck/capabilities.md
   - ideas/idea-mix/deck/frictions.md
   - ideas/idea-mix/deck/stimuli.md
```

#### `list` — デッキ一覧

```
使い方: /idea-mix-deck list [capability|friction|stimulus|all]

1. 指定されたデッキファイルを読み込み
2. カード一覧を表形式で表示:

   | # | concrete | abstract | 追加日 |
   |---|----------|----------|--------|
   | 1 | Claude Codeスキルが作れる | 手続き的知識の宣言的定義... | 2026-03-18 |
```

#### `abstract` — 再抽象化

```
使い方: /idea-mix-deck abstract [番号]

1. 指定されたカードを card-abstractor で再抽象化
2. 抽象度を上げ直す or 下げる
3. ユーザーの確認後に更新
```

#### `sync` — 外部ソースからの同期

```
使い方: /idea-mix-deck sync

1. pain-log/ から新しいエントリを摩擦カードに同期
   - 過去2週間の未同期エントリを抽出
   - 高頻度・未解決のものをカード化

2. daily-trend/, ios-daily/, TIL から刺激カードに同期
   - 構造的に面白いパターンを抽出
   - カードとして追加

3. 同期結果を表示
```

#### `prune` — 整理

```
使い方: /idea-mix-deck prune

1. 使用履歴を確認（ideas/idea-mix/ のフロントマターから）
2. 過去3ヶ月使われていないカードをリスト
3. ユーザーの確認後に削除 or アーカイブ
```

### デッキファイルフォーマット

```markdown
---
title: "能力カード"
type: capability
updated: 2026-03-18
card_count: 5
---

# 能力カード

## 1. 手続き的知識の宣言的定義+AI実行基盤
- **concrete**: Claude Codeのスキルシステムが作れる
- **abstract**: 手続き的知識を宣言的に定義しAIランタイムに実行させる基盤を構築できる
- **added**: 2026-03-18

## 2. モバイルネイティブ体験設計
- **concrete**: iOSアプリが作れる
- **abstract**: モバイルネイティブのインタラクティブ体験を設計・実装できる
- **added**: 2026-03-18

...
```

## PROOF

### Do / Don't

**Do**:
- ✅ 追加時に必ず抽象化を実施する
- ✅ 2層構造（concrete + abstract）を維持する
- ✅ 定期的に sync で外部ソースからカードを更新する

**Don't**:
- ❌ 具体的な記述だけのカードを許可する
- ❌ 過度に抽象的なカードを許可する
- ❌ ユーザーの確認なしにカードを削除する
