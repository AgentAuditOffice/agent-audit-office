# Summary / 集計

Generated / 作成: 2026-11-17T13:21:51 UTC  
Observation runs / 観測回数: 11 (2026-11-14T1759 – 2026-11-17T2126 JST)  
Reference time / 基準時刻: 2026-11-17 12:00:00 UTC (the 90-day payment window ends here / 過去 90 日の支払いはこの時刻まで)  
Registered agents on Base / Base の登録エージェント数: 1,205,898

## Grades / 格付け

| Grade / 格付け | Agents / 件数 |
|---|---:|
| A | 14 |
| B | 238 |
| C | 2,966 |
| Not rated (observed) / 対象外（観測した中で） | 7,533 |
| Not rated (no endpoint, not observed) / 対象外（接続先なし） | 1,195,147 |

Only agents graded A or B are listed individually ([ratings_AB.csv](ratings_AB.csv)). A・B のみ個別に掲載しています。

## Liveness (latest run) / 稼働率（最新の観測回）

Run / 観測回: 2026-11-17T2126 JST

| | Alive / 生存 | Judged / 判定対象 | Rate / 生存率 |
|---|---:|---:|---:|
| Endpoints / エンドポイント | 3,932 | 13,958 | 28.2% |
| Agents / エージェント | 3,218 | 10,138 | 31.7% |
| MCP | 485 | 4,200 | 11.5% |
| A2A | 631 | 2,285 | 27.6% |
| WEB | 2,748 | 7,297 | 37.7% |
| OASF | 68 | 176 | 38.6% |

## Observation record / 観測の記録

Every full observation run (v3 criteria). Scheduled three times a day (08:07 / 14:07 / 20:07 JST); actual start times vary because GitHub Actions scheduled runs were delayed or skipped and some were filled by manual runs. Rates are over judged endpoints. / v3 基準の全件の観測回。予定は 1 日 3 回（日本時間 8:07 / 14:07 / 20:07）だが、GitHub Actions の定期実行の遅延・欠落と手動実行による補完のため、実際の開始時刻は一定でない。生存率は判定対象のエンドポイントに対する割合。

| # | Start (JST) / 開始日時（日本時間） | Observation point / 観測時点 | Alive endpoints / 生存エンドポイント | Rate / 生存率 | Alive agents / 生存エージェント | Used for these grades / 今回の格付けに使用 |
|---:|---|:---:|---:|---:|---:|:---:|
| 1 | 2026-11-14 17:59 | 1 | 3,926 | 28.1% | 3,215 | ✓ |
| 2 | 2026-11-14 21:29 | 2 | 3,933 | 28.2% | 3,218 | ✓ |
| 3 | 2026-11-15 11:33 | 3 | 3,931 | 28.2% | 3,217 | ✓ |
| 4 | 2026-11-15 21:39 | 4 | 3,929 | 28.1% | 3,220 | ✓ |
| 5 | 2026-11-16 11:41 | 5 | 3,934 | 28.2% | 3,220 | ✓ |
| 6 | 2026-11-16 21:52 | 6 (merged / まとめ) | 3,935 | 28.2% | 3,221 | ✓ |
| 7 | 2026-11-16 22:32 | 6 (merged / まとめ) | 3,930 | 28.2% | 3,220 | ✓ |
| 8 | 2026-11-17 14:02 | 7 (merged / まとめ) | 3,929 | 28.2% | 3,218 | ✓ |
| 9 | 2026-11-17 14:15 | 7 (merged / まとめ) | 3,935 | 28.2% | 3,220 | ✓ |
| 10 | 2026-11-17 20:21 | 8 | 3,932 | 28.2% | 3,219 | ✓ |
| 11 | 2026-11-17 21:26 | 9 | 3,932 | 28.2% | 3,218 | ✓ |
| 12 | 2026-11-17 23:17 | 10 | 3,932 | 28.2% | 3,218 |  |

Runs that started within 60 minutes of the previous run share one observation point. An agent counts as alive at a merged point only if it was alive in every run of that point. / 前の回の開始から 60 分以内に始まった回は同じ観測時点にまとめた。まとめた時点では、その中の全回で生存した場合だけ生存とみなす。

## Feedback contamination / 評価の汚染率

| Excluded as / 除外の理由 | Records / 件数 | Share / 割合 |
|---|---:|---:|
| Tags miner-vouch / botcoin | 390,876 | 57.3% |
| Senders with 10+ in one hour (386 senders) / 1 時間に 10 件以上の送り手 | 136,557 | 20.0% |
| Mutual rating (57,009 owner pairs) / 相互評価 | 113,686 | 16.7% |
| Out-of-range values / 範囲外の値 | 1,716 | 0.3% |
| Self-rating / 自己評価 | 7 | 0.0% |
| **Total excluded / 除外合計** | **642,842** | **94.2%** |
| Healthy feedback kept / 健全な評価として残ったもの | 39,301 | 5.8% |

## Feedback scores by agent status / 状態別の評価（平均点・満点率）

| Status / 状態 | Agents / 体数 | Rated agents / 評価あり | Feedback records / 評価件数 | Scored / 点数として集計 | Mean score / 平均点 | Perfect-score rate / 満点率 |
|---|---:|---:|---:|---:|---:|---:|
| No endpoint / 接続先なし | 1,195,147 | 156,284 | 645,278 | 253,511 | 83.5 | 58.5% |
| Live (stable: A, B, C) / 生存（安定稼働: A・B・C） | 3,218 | 1,349 | 16,252 | 15,450 | 68.2 | 31.0% |
| Unstable or down / 不安定・停止 | 7,533 | 2,470 | 20,613 | 20,590 | 69.6 | 8.7% |

Scores are on a 0–100 scale; a perfect score is 100. Records tagged miner-vouch or botcoin and values outside 0–100 are not scored. "Live" and "unstable or down" follow the stability rule in methodology.md; "no endpoint" agents were never observed. / 点数は 0〜100、満点は 100。miner-vouch・botcoin のタグ付きと 0〜100 の範囲外の値は点数に含めない。「生存」「不安定・停止」は methodology.md の安定稼働の判定による。「接続先なし」は観測対象外。

## x402 payments (90 days to 2026-11-17 12:00:00 UTC) / x402 の支払い（基準時刻までの 90 日）

| Item / 項目 | Value / 値 |
|---|---:|
| Receiving wallets with x402 payments / x402 の支払いを受けたウォレット | 30 |
| Payments / 支払い件数 | 2,430 |
| Total amount (USDC) / 合計金額 | 10,850.25 |
| Share paid by owners or other registered agents / owner・他の登録エージェントからの支払いの割合 | 0.1% |
| Share received by the largest single wallet / 最大の 1 ウォレットの受取割合 | 99.8% |

See [methodology.md](../methodology.md) for definitions. / 定義は methodology.md を参照。
