# Summary / 集計

> **暫定（試運転）**: 観測回数が足りないため、安定稼働に必要な観測回数を一時的に下げて判定した結果です。本番の判定ではありません。
>
> **Provisional (trial run)**: there were not enough observation runs yet, so the minimum run count for "stable" was lowered temporarily. These are not final ratings.

Generated / 作成: 2026-11-14T12:55:21 UTC  
Observation runs / 観測回数: 2 (2026-11-14T1759 – 2026-11-14T2129 JST)  
Registered agents on Base / Base の登録エージェント数: 1,179,762

## Grades / 格付け

| Grade / 格付け | Agents / 件数 |
|---|---:|
| A | 14 |
| B | 238 |
| C | 2,963 |
| Not rated (observed) / 対象外（観測した中で） | 7,536 |
| Not rated (no endpoint, not observed) / 対象外（接続先なし） | 1,169,011 |

Only agents graded A or B are listed individually ([ratings_AB.csv](ratings_AB.csv)). A・B のみ個別に掲載しています。

## Liveness (latest run) / 稼働率（最新の観測回）

Run / 観測回: 2026-11-14T2129 JST

| | Alive / 生存 | Judged / 判定対象 | Rate / 生存率 |
|---|---:|---:|---:|
| Endpoints / エンドポイント | 3,933 | 13,958 | 28.2% |
| Agents / エージェント | 3,218 | 10,138 | 31.7% |
| MCP | 485 | 4,200 | 11.5% |
| A2A | 631 | 2,285 | 27.6% |
| WEB | 2,749 | 7,297 | 37.7% |
| OASF | 68 | 176 | 38.6% |

## Feedback contamination / 評価の汚染率

| Excluded as / 除外の理由 | Records / 件数 | Share / 割合 |
|---|---:|---:|
| Tags miner-vouch / botcoin | 390,874 | 57.7% |
| Senders with 10+ in one hour (386 senders) / 1 時間に 10 件以上の送り手 | 136,543 | 20.2% |
| Mutual rating (57,001 owner pairs) / 相互評価 | 113,668 | 16.8% |
| Out-of-range values / 範囲外の値 | 1,700 | 0.3% |
| Self-rating / 自己評価 | 7 | 0.0% |
| **Total excluded / 除外合計** | **642,792** | **94.9%** |
| Healthy feedback kept / 健全な評価として残ったもの | 34,263 | 5.1% |

## x402 payments (last 90 days) / x402 の支払い（過去 90 日）

| Item / 項目 | Value / 値 |
|---|---:|
| Receiving wallets with x402 payments / x402 の支払いを受けたウォレット | 30 |
| Payments / 支払い件数 | 2,556 |
| Total amount (USDC) / 合計金額 | 11,184.92 |
| Share paid by owners or other registered agents / owner・他の登録エージェントからの支払いの割合 | 0.1% |
| Share received by the largest single wallet / 最大の 1 ウォレットの受取割合 | 99.8% |

See [methodology.md](../methodology.md) for definitions. / 定義は methodology.md を参照。
