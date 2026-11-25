# Summary

Generated: 2026-11-24T13:05:15 UTC  
Observation runs: 20 (2026-11-17T2317 – 2026-11-24T2007 JST)  
Reference time: 2026-11-24 12:00:00 UTC (the 90-day payment window ends here)  
Registered agents on Base: 1,229,453

## Grades

| Grade | Agents |
|---|---:|
| A | 14 |
| B | 30 |
| C | 3,167 |
| Not rated (observed) | 7,540 |
| Not rated (no endpoint, not observed) | 1,218,702 |

Only agents graded A are listed individually ([ratings_A.csv](ratings_A.csv)).

Report #2 lists only Grade A agents individually. From this report, feedback exchanged within clusters and networks of agents that share endpoints across different owners is no longer counted as healthy feedback (see methodology, section 6). The number of records removed by this rule is shown in the feedback contamination table.

## Liveness (latest run)

Run: 2026-11-24T2007 JST

| | Alive | Judged | Rate |
|---|---:|---:|---:|
| Endpoints | 3,926 | 13,958 | 28.1% |
| Agents | 3,214 | 10,138 | 31.7% |
| MCP | 485 | 4,200 | 11.5% |
| A2A | 628 | 2,285 | 27.5% |
| WEB | 2,745 | 7,297 | 37.6% |
| OASF | 68 | 176 | 38.6% |

## Observation record

Every full observation run (v3 criteria) in this report's observation period. Runs are scheduled three times a day (08:07 / 14:07 / 20:07 JST). Until 2026-11-19 they were started by GitHub Actions schedules, which were sometimes delayed or skipped; from 2026-11-20 they are started by a Cloudflare Workers Cron. Rates are over judged endpoints.

| # | Start (JST) | Observation point | Alive endpoints | Rate | Alive agents | Used for these grades |
|---:|---|:---:|---:|---:|---:|:---:|
| 1 | 2026-11-17 23:17 | 1 | 3,932 | 28.2% | 3,218 | ✓ |
| 2 | 2026-11-18 13:10 | 2 | 3,934 | 28.2% | 3,220 | ✓ |
| 3 | 2026-11-18 22:50 | 3 | 3,911 | 28.0% | 3,197 | ✓ |
| 4 | 2026-11-19 13:06 | 4 | 3,929 | 28.1% | 3,217 | ✓ |
| 5 | 2026-11-19 23:19 | 5 | 3,931 | 28.2% | 3,227 | ✓ |
| 6 | 2026-11-20 12:23 | 6 | 3,929 | 28.1% | 3,216 | ✓ |
| 7 | 2026-11-20 14:07 | 7 | 3,927 | 28.1% | 3,216 | ✓ |
| 8 | 2026-11-20 20:07 | 8 | 3,946 | 28.3% | 3,231 | ✓ |
| 9 | 2026-11-21 08:07 | 9 | 3,928 | 28.1% | 3,215 | ✓ |
| 10 | 2026-11-21 14:07 | 10 | 3,927 | 28.1% | 3,213 | ✓ |
| 11 | 2026-11-21 20:07 | 11 | 3,927 | 28.1% | 3,213 | ✓ |
| 12 | 2026-11-22 08:07 | 12 | 3,931 | 28.2% | 3,216 | ✓ |
| 13 | 2026-11-22 14:07 | 13 | 3,925 | 28.1% | 3,212 | ✓ |
| 14 | 2026-11-22 20:07 | 14 | 3,929 | 28.1% | 3,215 | ✓ |
| 15 | 2026-11-23 08:07 | 15 | 3,926 | 28.1% | 3,212 | ✓ |
| 16 | 2026-11-23 14:07 | 16 | 3,930 | 28.2% | 3,216 | ✓ |
| 17 | 2026-11-23 20:07 | 17 | 3,928 | 28.1% | 3,215 | ✓ |
| 18 | 2026-11-24 08:07 | 18 | 3,931 | 28.2% | 3,217 | ✓ |
| 19 | 2026-11-24 14:07 | 19 | 3,928 | 28.1% | 3,214 | ✓ |
| 20 | 2026-11-24 20:07 | 20 | 3,926 | 28.1% | 3,214 | ✓ |

Runs that started within 60 minutes of the previous run share one observation point. An agent counts as alive at a merged point only if it was alive in every run of that point.

## Feedback contamination

| Excluded as | Records | Share |
|---|---:|---:|
| Tags miner-vouch / botcoin | 390,874 | 56.9% |
| Senders with 10+ in one hour (387 senders) | 136,581 | 19.9% |
| Mutual rating (57,011 owner pairs) | 113,690 | 16.6% |
| Out-of-range values | 1,719 | 0.3% |
| Self-rating | 7 | 0.0% |
| Within clusters or networks (2 networks, 862 agents) | 2,233 | 0.3% |
| **Total excluded** | **645,104** | **94.0%** |
| Healthy feedback kept | 41,440 | 6.0% |

## Feedback scores by agent status

| Status | Agents | Rated agents | Feedback records | Scored | Mean score | Perfect-score rate |
|---|---:|---:|---:|---:|---:|---:|
| No endpoint | 1,218,702 | 160,629 | 649,678 | 257,911 | 83.8 | 59.2% |
| Live (stable: A, B, C) | 3,211 | 1,346 | 16,240 | 15,438 | 68.2 | 31.0% |
| Unstable or down | 7,540 | 2,473 | 20,626 | 20,602 | 69.6 | 8.7% |

Scores are on a 0–100 scale; a perfect score is 100. Records tagged miner-vouch or botcoin and values outside 0–100 are not scored. "Live" and "unstable or down" follow the stability rule in methodology.md; "no endpoint" agents were never observed.

## x402 payments (90 days to 2026-11-24 12:00:00 UTC)

| Item | Value |
|---|---:|
| Receiving wallets with x402 payments | 31 |
| Payments | 2,339 |
| Total amount (USDC) | 8,936.50 |
| Share paid by owners or other registered agents | 0.1% |
| Share received by the largest single wallet | 99.4% |

### Third-party payments before the reference time (2026-11-24 12:00:00 UTC)

| Window | Payments | Total (USDC) | Receiving wallets | Receiving agents | Payers |
|---|---:|---:|---:|---:|---:|
| 30 days | 623 | 38.36 | 20 | 25 | 53 |
| 90 days | 1,885 | 8,931.07 | 30 | 40 | 266 |

Third-party payments exclude payers that are the agent's owner, another wallet of the same owner, or any registered agent's wallet (see methodology.md). The table above this one counts all payments. Receiving agents are agents alive in the latest observation run whose agentWallet received a payment (several agents can share one wallet).

See [methodology.md](https://github.com/AgentAuditOffice/agent-audit-office/blob/main/reports/2026-11-25-report-2/methodology.md) for definitions.
