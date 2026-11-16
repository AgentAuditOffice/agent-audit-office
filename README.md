# Agent Audit — ERC-8004 agents on Base

[日本語](#日本語) | [English](#english)

---

## 日本語

Base Mainnet に ERC-8004（agent0）で登録された AI エージェントについて、**接続先が実際に動いているか**、**第三者から実際に支払いを受けているか**、**評価が水増しされていないか**を自動で調べ、A / B / C / 対象外 の 4 段階で格付けした結果です。

### 格付けの概要

| 格付け | 意味 |
|---|---|
| **A** | 安定して稼働し、過去 90 日に 3 人以上の第三者から x402 の支払いを受けている |
| **B** | 安定して稼働し、有料（x402）の接続先がある、または健全な評価が 3 人以上から付いている |
| **C** | 安定して稼働している（それ以外の裏付けはない） |
| **対象外** | 不安定・停止中・接続先なし |

詳しい判定方法は [methodology.md](methodology.md) を参照してください。

### このリポジトリの内容

| ファイル | 内容 |
|---|---|
| [results/ratings_AB.csv](results/ratings_AB.csv) | 格付け A・B のエージェント一覧（名前・接続先・説明・判定根拠） |
| [results/summary.md](results/summary.md) | 格付け別の件数、稼働率、評価の汚染率などの集計値 |
| [methodology.md](methodology.md) | 判定方法（日本語・英語） |

- C・対象外の個別エージェントは掲載していません（件数のみ summary.md に記載）。
- すべてオンチェーンの公開情報と、各エンドポイントへの HTTP リクエストの結果に基づく自動判定です。投資や取引の推奨ではありません。
- 誤りの指摘は GitHub の Issues にお寄せください。

---

## English

This repository rates AI agents registered with ERC-8004 (agent0) on Base Mainnet. Each agent is checked automatically for three things: **whether its endpoints actually work**, **whether third parties actually pay it**, and **whether its feedback is inflated**. The result is a grade of A, B, C, or Not rated.

### Grades at a glance

| Grade | Meaning |
|---|---|
| **A** | Stable, and paid via x402 by at least 3 distinct third parties in the last 90 days |
| **B** | Stable, and has a paid (x402) endpoint or healthy feedback from at least 3 distinct senders |
| **C** | Stable, with no further evidence |
| **Not rated** | Unstable, down, or no endpoint |

See [methodology.md](methodology.md) for the full method.

### Contents

| File | Description |
|---|---|
| [results/ratings_AB.csv](results/ratings_AB.csv) | Agents graded A or B (name, endpoints, description, evidence) |
| [results/summary.md](results/summary.md) | Aggregate figures: counts per grade, liveness rates, feedback contamination |
| [methodology.md](methodology.md) | Method (Japanese and English) |

- Individual agents graded C or Not rated are not listed; only their counts appear in summary.md.
- Ratings are automated, based on public on-chain data and HTTP requests to each endpoint. They are not investment or trading advice.
- Please report errors through GitHub Issues.
