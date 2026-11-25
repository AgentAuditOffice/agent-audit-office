# Rating Methodology

Scope: ERC-8004 (agent0) agents registered on Base Mainnet.  
From Report #3: also Ethereum (chainId 1) and BNB Chain (chainId 56); see section 7.  
Script: `score.py` (all thresholds live in the `CONFIG` block at the top).  
Japanese version: [methodology.ja.md](methodology.ja.md)

## 1. Data sources

| Data | Source | Script |
|---|---|---|
| Agent registry (owner, agentWallet, endpoints, name, description) | The Graph, agent0 subgraph | `fetch_agents.py` |
| Endpoint liveness | One HTTP request per endpoint, three times a day (08:07 / 14:07 / 20:07 JST) on GitHub Actions | `probe.py` |
| x402 payments | x402 Base subgraph on The Graph Network (`Cb56epg3EvQ6JRpPfknbkM54QxpzTvLa7mwKNQQfUyoj`) | `fetch_payments.py` |
| Feedback | agent0 subgraph, `Feedback` entity | `fetch_feedback.py` |

The x402 subgraph ID comes from third-party documentation (Graph Advocate), not The Graph's official docs. A sample of 25 payments was checked against Base public RPC transaction receipts, and recipient, amount and payer matched in all 25 (`verify_payments.py`).

## 2. Liveness per observation (v3 criteria)

| Kind | Request | Counted as alive when |
|---|---|---|
| MCP | JSON-RPC `initialize` POST | 2xx and the body contains a JSON-RPC `result` |
| A2A | GET `/.well-known/agent.json` (or the URL as-is if it already points to a card) | 2xx and a JSON body with `name` |
| Web / OASF | GET | 2xx |

- **alive_paid**: HTTP 402 with x402 payment terms (`PAYMENT-REQUIRED` header, or `x402Version` / `accepts` in the body). Counted as alive (paid).
- **auth_required** (401/403) and **uncertain** (400, 405, or 402 without terms) are not counted as alive.
- **dead**: 404, other 4xx, 5xx, 2xx without a valid body, timeout, DNS error, or connection failure.
- Not judged: github.com targets (skipped), internal addresses (never contacted), invalid URLs.
- An agent is alive in a run if any of its endpoints is alive.
- Bodies are parsed as JSON (max 64KB) only. They are never stored or executed.

## 3. Definitions

**Observation point** — a full v3 run that starts **within 60 minutes** of the previous run's start is merged into the same observation point, so a chain of runs each within 60 minutes of the last becomes one point. At a merged point, an agent counts as alive **only if it was alive in every run** of that point (for two merged runs, both must be alive). This keeps closely spaced runs from being counted twice.

**Stable** — the agent was observed at **at least 6 observation points** and was alive at least **7/9 (≈77.8%)** of them.

- The plan is 7 of 9 points. Scheduled GitHub Actions runs can be delayed or skipped, so both a minimum number of points and a ratio are required.
- alive_paid counts as alive; auth_required and uncertain do not.
- The denominator is the number of points at which the agent was observed.

**Reference time** — the first report's reference time is **2026-11-17 12:00 UTC**. The 90-day payment window is counted back from it (2026-08-19 12:00 to 2026-11-17 12:00 UTC), and later payments are excluded. The reference time applies only to the payment window; observations include the run that started at 21:26 JST (12:26 UTC) on 2026-11-17.

**Third-party payment** — an x402 payment in the last 90 days whose payer is none of the following (grade A requires **at least 3 distinct** such payers):
- the agent's owner
- the agentWallet of another agent with the same owner
- the agentWallet of any other registered agent (checked against all ~1.18M registrations)

Payments that do match one of these are "suspicious", and their share of the total amount received is computed.

**Healthy feedback** — feedback from **at least 3 distinct senders** remains after removing:
1. records whose `tag1` or `tag2` is `miner-vouch` or `botcoin`
2. values outside 0–100
3. all feedback from any sender that ever sent 10 or more within one hour
4. mutual-rating pairs: if A's side rates B and B's side rates A, both directions are removed
5. self-rating: the sender is the target's owner or an agentWallet of the same owner

Mutual and self-rating are evaluated **per owner**. A sender address maps to the owners it represents, either as the owner itself or as an agentWallet. This also catches mutual rating routed through different agents of the same owner. Excluding self-rating is not in the original table; it was added to match the intent of "distinct third parties" and can be switched off with `CONFIG["exclude_self"]`.

## 4. Grades (evaluated top to bottom; the first match wins)

| Grade | Condition |
|---|---|
| **A** | Stable + third-party payments from **at least 3 distinct payers** in the last 90 days + suspicious payments are less than 50% of the amount received |
| **B** | Stable + (an endpoint judged alive_paid during the observation period, or healthy feedback) |
| **C** | Stable only |
| **Not rated** | Everything else (unstable, dead, or no endpoint) |

"An endpoint judged alive_paid" means at least one observation run judged one of the agent's endpoints alive_paid.

**Agents sharing a wallet** are treated as one business for payments. If several agents with the same agentWallet qualify for A, only one gets A and the rest get B. The representative is chosen by uptime, then number of alive runs, then number of alive_paid runs, then registration age (smallest agentId).

**Notes (`note` column in `results/ratings_A.csv`)** — A-rated agents that sit exactly at a threshold get a note.

- Alive at exactly the minimum number of observation points (e.g. 7 of 9). If the agent would not be stable with a 90-minute merge window, the note says so.
- Exactly 3 third-party payers.

## 5. Known limitations

- **Shared wallets**: payments are recorded per receiving wallet. Agents sharing an agentWallet are treated as one business and only one of them can get A (grades B and below still use each agent's own record).
- **Self-dealing coverage**: only matches against registered owners and agentWallets. Self-dealing through unregistered wallets is not detected.
- **Non-x402 payments**: the x402 subgraph only records facilitator-settled payments. Plain USDC transfers are not included.
- **Observation network**: GitHub Actions connects from data-center IPs, and some sites (e.g. basescan.org, x.com) answer 403, which is recorded as auth_required.
- **github.com endpoints** are not judged.
- **Uneven observation times (first report)**: observations are scheduled three times a day (08:07 / 14:07 / 20:07 JST). During the first report's observation period, however, GitHub Actions scheduled runs were delayed by up to about 3.5 hours or skipped entirely. Some missing runs were filled with manual runs, so observation times and intervals are uneven. Closely spaced runs are merged into observation points, and stability is judged as "at least 6 observation points and alive at 7/9 or more of them". The actual observation times, per-run figures, and which runs were merged are in the "Observation record" section of `results/summary.md`.
- **Change in how observations are started (from 2026-11-20)**: from 2026-11-20, observation runs are started by a Cloudflare Workers Cron (23:07 / 05:07 / 11:07 UTC = 08:07 / 14:07 / 20:07 JST), which triggers the GitHub Actions workflow. This replaces GitHub Actions scheduled runs to reduce delays and skipped runs. What each run requests and how it judges are unchanged. All observations in the first report (up to the run that started at 21:26 JST on 2026-11-17) used the previous method.
- **The trial runs (2026-11-14)** had only 1–2 observation runs, so no agent met the minimum of 6 observation points. They were run with `--min-runs` to lower the minimum temporarily. The production rating runs after the observation period with the minimum kept at 6 points.

## 6. Changes from Report #2

### 6.1 Confirmed: Report #2 observation period and reference time

These were fixed and published before the data is final.

| Item | Report #2 |
|---|---|
| Observations | Runs that started after 2026-11-17 21:26 JST and no later than 2026-11-24 21:00 JST. They do not overlap with Report #1's observations |
| Reference time | 2026-11-24 12:00 UTC (21:00 JST) |
| 90-day payment window | 2026-08-26 12:00 to 2026-11-24 12:00 UTC |
| 30-day payment window (for the summary) | 2026-10-25 12:00 to 2026-11-24 12:00 UTC |
| Planned publication | 2026-11-25 22:00 JST |
| Stability rule | Same as Report #1 (runs that start within 60 minutes of the previous run form one observation point; at least 6 observation points and alive at 7/9 or more of them) |

Observations are scheduled three times a day (08:07 / 14:07 / 20:07 JST); from 2026-11-20 they are started by a Cloudflare Workers Cron (section 5). The observation cutoff (2026-11-24 21:00 JST) is the same moment as the reference time, so Report #2 uses no observations that started after the reference time.

### 6.2 Confirmed: excluding feedback within clusters and networks (applies from Report #2)

We found many agents with different owners that share the same endpoints and rate one another, which let them meet the "healthy feedback" condition. From Report #2, such feedback is excluded from healthy feedback under the following rules.

**URL normalization** — endpoint URLs are compared after trimming whitespace, lowercasing, and removing the query, fragment, default port and trailing slash.

**Cluster** — a group of agents that meets either condition:

- **URL cluster**: the same normalized endpoint URL is registered by **at least 3** agents with different owners.
- **Host cluster**: the same endpoint host (including different paths on that host) is registered by **at least 3** agents with different owners. Shared hosts where many unrelated projects live (code-hosting sites, social networks, block explorers, IPFS gateways and similar) are excluded.

**Network** — two host clusters are linked when the members of one sent **at least 5** feedback records to the agents of the other. Linked groups of two or more clusters form a network. Only feedback that remains after the other filters (items 1–5 under healthy feedback in section 3) is counted. Hosts of URLs that are often entered as placeholders, such as the front pages of major websites, are not used to detect networks.

**Members** — the owners and agentWallets of the agents in a cluster or network.

**Exclusion rule** — feedback whose sender is a member of the same cluster or the same network as the rated agent is removed from healthy feedback. The "at least 3 distinct senders" condition is checked after this removal.

**Sensitivity analysis (on Report #1 data)** — we varied the number of feedback records needed to link two clusters. The test set was the 225 of Report #1's 238 B agents whose grade rested on healthy feedback.

| Records needed to link | Networks | Clusters in networks | Agents | B agents losing their basis |
|---:|---:|---:|---:|---:|
| 3 | 4 | 16 | 1,046 | 208 |
| **5 (adopted)** | **2** | **12** | **862** | **208** |
| 10 | 2 | 12 | 862 | 208 |
| Reference: within clusters only (no networks) | – | – | – | 1 |

From 3 to 10 records, the same 208 agents lose their healthy-feedback basis, so the result is stable. Excluding feedback only within single clusters removes just 1, so network-level exclusion is needed to catch agents rating each other across clusters. Applied to Report #1 data, B would fall from 238 to 30 agents, and all 208 agents removed from B would be graded C.

This rule was not applied to Report #1, and Report #1's grades are unchanged.

### 6.3 Planned or under consideration (not yet confirmed)

- **Additional condition for grade A**: third-party payments must span **at least 3 distinct days** (UTC calendar days). This keeps payments bunched into a short burst from being enough for an A.
- **New note (payer concentration)**: add a note when the top payer accounts for more than 50% of the amount received.
- **New note (new wallets)**: add a note when more than 50% of payers used wallets created within 7 days before their first payment to the agent. A wallet's creation date is the earliest of its first self-sent transaction, its smart-wallet deployment, and its first x402 payment; merely receiving funds does not count.
- **Under consideration for grade A**: require at least one third-party payment within 30 days before the reference time. This keeps agents that are no longer used from earning an A on past payments alone.

None of these planned changes were applied to Report #1's grades or notes.

## 7. Report #3 and later: Ethereum and BNB Chain

Published 2026-11-21, before any Report #3 data is final. These rules apply from Report #3. Report #1 and Report #2 are not affected. A Japanese translation is in [methodology.ja.md](methodology.ja.md); if the two differ, this English version takes precedence.

### 7.1 Changes from Report #2 (all chains, Base included)

From Report #3, these changes also apply to Base. Report #2 is not affected.

1. **Clusters and networks are detected across all three chains together** (7.7.1). This was decided on principle. Its effect on Base was not measured before the rule was fixed.
2. **Mutual rating and self-rating are matched against all three registries combined** (7.7.2).
3. **Two host lists with separate effects** (7.8):
   - **Not judged:** basescan.org, bscscan.com, etherscan.io and 8004scan.io (each with its `www.` form) are added. They are still probed, but their results are treated as `excluded` in grading, the same as github.com. They are also not used for host clusters.
   - **Shared hosting:** the AWS AgentCore endpoint `bedrock-agentcore.us-east-1.amazonaws.com` is added. It is still probed and judged, but it is not used for host clusters.
4. **Revoked feedback is excluded** from healthy feedback (7.5).

### 7.2 Scope

From Report #3, ERC-8004 (agent0) agents on three chains are rated:

| Chain | chainId | agent0 subgraph (The Graph Network) |
|---|---:|---|
| Base | 8453 | `43s9hQRurMGjuYnC1r2ZwS6xSQktbFyXMPMqGKUFJojb` |
| Ethereum | 1 | `FV6RR6y13rsnCxBAicKuQEwDp8ioEGiNaWaZUmvr1F8k` |
| BNB Chain | 56 | `D6aWqowLkWqBgcqmpNKXuNikPkob24ADXCciiP8Hvn1K` |

Subgraph IDs come from the agent0 subgraph repository (github.com/agent0lab/subgraph) and The Graph's documentation.

- Each chain is rated separately. An agent is identified by `chainId:agentId`. If the same project registers on several chains, each registration is rated on its own, using only that chain's data, except for the cross-chain checks in 7.7.
- Results are reported per chain. Chains are not merged into one ranking.

### 7.3 Report #3 schedule

These values are fixed now, before the data is final.

| Item | Report #3 |
|---|---|
| Reference time (as_of) | 2026-12-01 12:00 UTC (21:00 JST) |
| Base observations | Runs that started after 2026-11-24 21:00 JST and no later than 2026-12-01 21:00 JST (no overlap with Report #2) |
| Ethereum / BNB Chain observations | From the first scheduled GitHub Actions run that included these chains (scheduled for 2026-11-21 20:07 JST, started by the Cloudflare Workers Cron) to runs that started no later than 2026-12-01 21:00 JST |
| 90-day payment window (Base only) | 2026-09-02 12:00 to 2026-12-01 12:00 UTC |
| 30-day payment window (Base only, for the summary) | 2026-11-01 12:00 to 2026-12-01 12:00 UTC |
| Planned publication | 2026-12-02 22:00 JST |

- Observation points expected by the cutoff if no run is missed (three runs a day at 08:07 / 14:07 / 20:07 JST): **Base 21, Ethereum 31, BNB Chain 31**. Missed runs reduce these numbers. The stability rule already requires at least 6 points, so a few missed runs do not change how stability is judged.
- The one-off Ethereum and BNB Chain runs on 2026-11-21 at 14:32 and 14:33 JST are excluded. They were started by hand from a development environment through a proxy, not from GitHub Actions.
- Ethereum and BNB Chain are probed in the same GitHub Actions run as Base, right after it, and each chain's run is a separate file. Runs are merged into observation points within each chain only.

### 7.4 Data sources

| Data | Base | Ethereum | BNB Chain |
|---|---|---|---|
| Agent registry | agent0 subgraph (7.2) | agent0 subgraph (7.2) | agent0 subgraph (7.2) |
| Endpoint liveness | `probe.py` | `probe.py` via `probe_chain.py` (same code, criteria and safety rules) | same as Ethereum |
| x402 payments | x402 Base subgraph (`Cb56epg3…`), sample-checked against RPC receipts | **None available** | **None available** |
| Feedback | agent0 `Feedback` entity | agent0 `Feedback` entity (same fields as Base) | agent0 `Feedback` entity (same fields as Base) |

**Candidate lists (fixed for Report #3).** Ethereum and BNB Chain candidates are the agents that, on 2026-11-21 (about 05:30 UTC), had at least one MCP / A2A / Web / OASF endpoint or `x402Support = true`. This is the same rule as Base. These lists stay fixed through Report #3, so agents registered later are not observed in Report #3. The Base list is also fixed. Any refresh will be announced for a later report.

### 7.5 Rules that apply on every chain

These rules are the same as for Base, and each is applied to one chain's data. The only exceptions are the cross-chain checks in 7.7 and the host lists in 7.8.

- liveness per observation (v3 criteria, section 2), with the host lists in 7.8
- a run counts as a full run with at least 1,000 endpoint rows; Ethereum and BNB Chain each have about 2,700–2,900 rows per run
- observation points (runs within 60 minutes merged; at a merged point, alive only if alive in every run of that point)
- **Stable**: at least 6 observation points, and alive at 7/9 or more of them
- **Healthy feedback**: at least 3 distinct senders remain after removing:
  0. **revoked feedback** (`isRevoked = true` in the data fetched for the report). New from Report #3, on all chains.
  1. records tagged `miner-vouch` or `botcoin`
  2. values outside 0–100
  3. all feedback from senders with 10 or more records in one hour
  4. mutual rating (7.7.2)
  5. self-rating (7.7.2)

  Filter 0 runs first, so revoked records also do not count toward the burst rule (filter 3) or mutual rating (filter 4).
- exclusion of feedback within clusters and networks (section 6.2: 3 owners for a cluster, 5 feedback records to link two clusters), detected across all chains (7.7.1)

### 7.6 Grades on Ethereum and BNB Chain

Evaluated top to bottom; the first match wins.

| Grade | Condition |
|---|---|
| **A** | **Not available on this chain.** Grade A requires verified x402 payments from at least 3 distinct third parties, and there is no verified x402 payment data for this chain (7.9) |
| **B** | Stable + (an endpoint judged alive_paid during the observation period, or healthy feedback) |
| **C** | Stable only |
| **Not rated** | Everything else (unstable, dead, or no judged endpoint) |

- The B, C and Not rated conditions are the same as on Base.
- No substitute for payment data is used. Payments on Base are never counted for an Ethereum or BNB Chain agent, even when both use the same wallet address. Neither `x402Support = true` nor an HTTP 402 response is treated as a payment.
- "alive_paid" means the endpoint answered with x402 payment terms, as on Base. The probe does not check which network the terms name.

**Presentation.** Only Grade A agents are named individually.

- Report #3 shows one aggregate table per chain: the counts of B, C and Not rated, with A shown as **"not available"** and the line "Grade A is not available on this chain (no verified x402 payment data)".
- No Ethereum or BNB Chain agent is named. The API returns `not_listed` for these agents.
- **Fallback:** a chain with fewer than 6 observation points at the cutoff is left out of Report #3. The report says so, and the chain starts in a later report.

**Base in Report #3.** Base grades A to C follow the Report #2 rules, with the changes listed in 7.1. Planned items in section 6.3 apply only if they are confirmed and published before Report #3 data is final.

### 7.7 Cross-chain checks

1. **Clusters and networks: all three chains together.**
   - URL and host clusters count owners across Base, Ethereum and BNB Chain.
   - A cluster's members are the owners and agentWallets of its agents on every chain.
   - Network links count feedback on any chain.
   - Feedback is excluded when its sender is a member of the same cluster or network as the rated agent, whichever chain the sender's own agent is on.
   - Agent IDs include the chainId, so the same agentId on two chains is never mixed up.
2. **Mutual rating and self-rating: all three registries combined.** A sender address is mapped to every owner it represents, as an owner or as an agentWallet, on any of the three chains. Both the mutual-rating pairs (filter 4) and the self-rating check (filter 5) use this combined map. Mutual-rating pairs are found from the feedback on all three chains. For example, if owner X rates owner Y on Ethereum and Y rates X on Base, the pair is mutual on both chains.
3. **One A per shared wallet: per chain.** Only agents on the same chain are grouped by agentWallet. In Report #3 only Base can have Grade A, so this has no cross-chain effect. It will be revisited when a second chain has payment data.

### 7.8 Host lists

**How it works up to Report #2.** There are two separate lists:

- **Not probed:** github.com and its subdomains. The probe never requests these endpoints, including redirect targets, and records them as `excluded` (not judged). github.com is also on the shared-host list below.
- **Shared hosts** (section 6.2): code hosting, social networks, block explorers, IPFS gateways and similar. These hosts are still probed and judged as usual. They are only left out when **host** clusters are built, and the host must match exactly. URL clusters (the same full URL from 3 or more owners) still include them.

basescan.org and etherscan.io are on the shared-host list only, so up to Report #2 they are probed and judged.

**From Report #3 (all chains, Base included)**, three host groups:

| Group | Hosts | Probed? | Judged in grading? | Used for host clusters? |
|---|---|---|---|---|
| **Not judged** (not the agent's own service) | github.com (existing), basescan.org, www.basescan.org, bscscan.com, www.bscscan.com, etherscan.io, www.etherscan.io, 8004scan.io, www.8004scan.io | Yes, except github.com (unchanged) | **No.** Results are treated as `excluded`, the same as github.com | No |
| **Shared hosting** | the existing shared-host list, plus `bedrock-agentcore.us-east-1.amazonaws.com` | Yes | Yes | No |
| Everything else | — | Yes | Yes | Yes |

- **Not judged** covers the listed host **and its subdomains** (e.g. api.etherscan.io).
  - The probe keeps requesting these hosts, so the raw observation data stays complete. Report #2's observations are not affected.
  - In grading, a result is treated as `excluded` when its request URL or its final URL after redirects is on one of these hosts.
  - An agent whose only endpoints are on these hosts has no judged endpoint and is Not rated.
- **Shared hosting** matches the exact host only, as today.
- **URL clusters are unchanged.** As today, the same full URL from 3 or more owners forms a URL cluster on any host.
- The placeholder hosts of section 6.2 are unchanged: they can form clusters but are not used to link networks.

### 7.9 Enabling Grade A on another chain

Grade A becomes available on Ethereum or BNB Chain only after all of the following:

1. A source of x402 payments on that chain is identified, giving payer, recipient, amount and time per payment.
2. A sample of at least 25 payments matches that chain's transaction receipts (recipient, amount, payer), as was done for Base (`verify_payments.py`).
3. The source and the sample result are published in `methodology.md` before the data of the first report that uses them is final.

For BNB Chain, step 1 must also say which settlement methods count as x402 payments. BNB Chain's main stablecoins do not support EIP-3009. Facilitators there settle through wrapped tokens or relay contracts, not through the same contract call as on Base.

### 7.10 Known limitations (Ethereum and BNB Chain)

- **No Grade A** until 7.9 is met. The best available grade is B.
- **Feedback is dominated by high-volume senders.** The filter for senders with 10 or more in one hour removes about 96% of Ethereum and about 98% of BNB Chain feedback, compared with about 20% on Base. Healthy feedback is therefore a small share of all feedback on these chains.
- **Smart-contract wallets across chains.** The cross-chain checks (7.7) treat one address as one holder on every chain. That holds for ordinary wallets (EOAs). A smart-contract wallet can exist at the same address on two chains under different control. Such a wallet could be wrongly linked across chains, and feedback from it could be excluded by mistake.
- **Fixed candidate list.** Agents registered after 2026-11-21 are not observed in Report #3.
- **Observation network.** Same as Base: observations run from GitHub Actions data-center IPs.
- **Endpoints on not-judged hosts** (7.8). Many Ethereum and BNB Chain agents list only a github.com OASF link. These agents have no judged endpoint and are Not rated (no judged endpoint), not dead.

### 7.11 Disclosure: results seen before these rules were written

The following Ethereum and BNB Chain results were seen before these rules were written. No grades, stability results or per-agent healthy-feedback counts were computed. The effect of the Report #3 changes on Base was not measured.

- Registry counts from the subgraphs. Ethereum: 56,722 agents, 1,093 with an MCP or A2A endpoint, 4,907 with `x402Support = true`, 2,703 with at least one feedback record. BNB Chain: 420,583 agents, 1,386 with an MCP or A2A endpoint, 14,427 with `x402Support = true`, 4,483 with at least one feedback record.
- Candidate counts. Ethereum: 6,154 candidates (1,541 with an endpoint; MCP 1,047, A2A 152, Web 1,063, OASF 475). BNB Chain: 16,120 candidates (1,958 with an endpoint; MCP 405, A2A 1,292, Web 816, OASF 411).
- One manual liveness run per chain on 2026-11-21 at 14:32 and 14:33 JST, excluded from Report #3 (7.3). Ethereum: endpoint liveness 45.6% (1,051 of 2,304 judged), agent liveness 87.0% (995 of 1,144), 0 alive_paid. BNB Chain: endpoint liveness 74.1% (1,853 of 2,499), agent liveness 72.6% (1,327 of 1,829), 2 alive_paid endpoints. The tables shown with these runs also gave per-kind liveness and error types. Why agents with an endpoint were not judged: Ethereum 390 had only github.com endpoints, 4 had invalid URLs and 3 had internal addresses; BNB Chain 114, 8, 4 and 3 (both blocked and github.com).
- Feedback aggregates after filters 1–5 and network exclusion (Report #2 settings, without the revoked filter), using all feedback to 2026-11-21. Ethereum: 37,723 records, 95.9% excluded (burst senders 95.7%), 0 revoked. BNB Chain: 30,162 records, 98.4% excluded (burst senders 98.1%), 1 revoked. Base: 166 revoked records out of 683,639. Also the most frequent tags and the top-5 senders' share of records.
- Cross-chain overlap counts: shared owners, shared hosts, shared agentWallets, host clusters per chain and jointly. The most-shared hosts per chain were looked at when choosing the host lists in 7.8.

### 7.12 Summary of decisions (2026-11-21)

| # | Topic | Decision |
|---|---|---|
| 1 | Cluster and network detection | All three chains together, from Report #3; effect on Base not measured before publication |
| 2 | Mutual and self-rating | All three registries combined; smart-contract wallet caveat under 7.10 |
| 3 | One A per shared wallet | Per chain for now |
| 4 | Host lists | Not judged: basescan.org, bscscan.com, etherscan.io, 8004scan.io (with `www.`); probed, but treated as `excluded` in grading. Shared hosting: AWS AgentCore endpoint. All chains, from Report #3 |
| 5 | Candidate lists | Fixed as of 2026-11-21 through Report #3 |
| 6 | Presentation | Per-chain aggregate table, A shown as "not available"; a chain with fewer than 6 points is left out |
| 7 | Enabling A on another chain | 7.9 as written |
| 8 | Revoked feedback | Excluded on all chains from Report #3; Report #2 unchanged |

### 7.13 Implementation

`score.py` has one flag for each new rule in `CONFIG`: `cross_chain_clusters`, `cross_chain_addresses`, `r3_host_groups` and `exclude_revoked_feedback`. All four are off by default, so Report #1 and Report #2 reproduce byte for byte. `score.py --r3-rules on` turns all four on, and `--chain base|eth|bsc` picks the chain to grade. `probe.py` is unchanged.
