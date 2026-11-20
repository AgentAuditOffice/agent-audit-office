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
| [results/ratings_A.csv](results/ratings_A.csv) | Agents graded A (name, endpoints, description, evidence, notes) |
| [results/summary.md](results/summary.md) | Aggregate figures: counts per grade, liveness rates, feedback contamination, payments |
| [methodology.md](methodology.md) | Method (Japanese and English) |
| [results/ratings_A.png](results/ratings_A.png) | Grade A list (image) |

- Only Grade A agents are listed individually. Agents graded B, C or Not rated appear only as counts in summary.md.
- Ratings are automated, based on public on-chain data and HTTP requests to each endpoint. They are not investment or trading advice.
- Please report errors through GitHub Issues or to contact@agentauditoffice.com.

## API and MCP

Free and read-only, with no sign-up, API key or cookies. Every response includes the report, its reference time (`as_of`), the methodology link and the license.

REST API (`https://api.agentauditoffice.com`):

| Request | Returns |
|---|---|
| `GET /v1/agents/{chainId}:{agentId}` (e.g. `/v1/agents/8453:45250`) | Grade A details, or `"status": "not_listed"` |
| `GET /v1/grade-a` | All Grade A agents |
| `GET /v1/summary` | Report figures |

Only Grade A agents are listed individually. Any other valid ID returns `not_listed`; not being listed does not mean an agent is untrustworthy. Malformed IDs return HTTP 400. Responses are cached and requests are rate-limited per IP.

MCP server (Streamable HTTP) at `https://api.agentauditoffice.com/mcp`, with read-only tools `lookup_agent`, `list_grade_a` and `get_report_summary`:

```json
{
  "mcpServers": {
    "agent-audit-office": { "type": "http", "url": "https://api.agentauditoffice.com/mcp" }
  }
}
```

## License

The data and documents are licensed under [CC BY 4.0](LICENSE). Please credit "Agent Audit Office (AAO)".

日本語版: [README.ja.md](README.ja.md)
