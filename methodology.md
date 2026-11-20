# Rating Methodology

Scope: ERC-8004 (agent0) agents registered on Base Mainnet.  
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
