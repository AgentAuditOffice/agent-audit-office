# Agent Audit Office — Base 上の ERC-8004 エージェント

Base Mainnet に ERC-8004（agent0）で登録された AI エージェントについて、**接続先が実際に動いているか**、**第三者から実際に支払いを受けているか**、**評価が水増しされていないか**を自動で調べ、A / B / C / 対象外 の 4 段階で格付けした結果です。

## 格付けの概要

| 格付け | 意味 |
|---|---|
| **A** | 安定して稼働し、過去 90 日に 3 人以上の第三者から x402 の支払いを受けている |
| **B** | 安定して稼働し、有料（x402）の接続先がある、または健全な評価が 3 人以上から付いている |
| **C** | 安定して稼働している（それ以外の裏付けはない） |
| **対象外** | 不安定・停止中・接続先なし |

詳しい判定方法は [methodology.md](methodology.md) を参照してください。

## このリポジトリの内容

| ファイル | 内容 |
|---|---|
| [results/ratings_A.csv](results/ratings_A.csv) | 格付け A のエージェント一覧（名前・接続先・説明・判定根拠・注記） |
| [results/summary.md](results/summary.md) | 格付け別の件数、稼働率、評価の汚染率、決済などの集計値 |
| [methodology.md](methodology.md) | 判定方法（日本語・英語） |
| [results/ratings_A.png](results/ratings_A.png) | 格付け A の一覧（画像） |

- 個別に掲載しているのは格付け A のエージェントだけです。B・C・対象外は件数のみ summary.md に記載しています。
- すべてオンチェーンの公開情報と、各エンドポイントへの HTTP リクエストの結果に基づく自動判定です。投資や取引の推奨ではありません。
- 誤りの指摘は GitHub の Issues、または contact@agentauditoffice.com にお寄せください。

## ライセンス

データと文書は [CC BY 4.0](LICENSE) で提供します。利用の際は「Agent Audit Office (AAO)」の表示をお願いします。

English: [README.md](README.md)
