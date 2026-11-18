# Agent Audit Office — ERC-8004 agents on Base

This repository rates AI agents registered with ERC-8004 (agent0) on Base Mainnet. Each agent is checked automatically for three things: **whether its endpoints actually work**, **whether third parties actually pay it**, and **whether its feedback is inflated**. The result is a grade of A, B, C, or Not rated.

## Grades at a glance

| Grade | Meaning |
|---|---|
| **A** | Stable, and paid via x402 by at least 3 distinct third parties in the last 90 days |
| **B** | Stable, and has a paid (x402) endpoint or healthy feedback from at least 3 distinct senders |
| **C** | Stable, with no further evidence |
| **Not rated** | Unstable, down, or no endpoint |

See [methodology.md](methodology.md) for the full method.

## Contents

| File | Description |
|---|---|
| [results/ratings_AB.csv](results/ratings_AB.csv) | Agents graded A or B (name, endpoints, description, evidence) |
| [results/summary.md](results/summary.md) | Aggregate figures: counts per grade, liveness rates, feedback contamination, payments |
| [methodology.md](methodology.md) | Method (Japanese and English) |
| [results/ratings_A.png](results/ratings_A.png) | Grade A list (image) |

- Individual agents graded C or Not rated are not listed; only their counts appear in summary.md.
- Ratings are automated, based on public on-chain data and HTTP requests to each endpoint. They are not investment or trading advice.
- Please report errors through GitHub Issues or to contact@agentauditoffice.com.

## License

The data and documents are licensed under [CC BY 4.0](LICENSE). Please credit "Agent Audit Office (AAO)".

日本語版: [README.ja.md](README.ja.md)
