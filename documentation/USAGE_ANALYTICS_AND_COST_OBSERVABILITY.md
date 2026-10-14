# 08 — Usage Analytics & Spend Observability

Flit provides real-time, auditable metrics on LLM consumption, token flow, latency percentiles, and calculated inference spend across your enterprise.

---

## Key Metrics & Observability Dimensions

| Metric | Measurement Unit | Description |
| :--- | :--- | :--- |
| **Total Requests** | Count | Aggregated volume of API completions, chat turns, and document extractions. |
| **Input (Prompt) Tokens** | Tokens | Volume of text, attachments, and system instructions ingested by models. |
| **Output (Completion) Tokens** | Tokens | Volume of generated tokens emitted by models. |
| **Calculated Spend** | USD ($) | Monetary cost calculated per turn based on exact upstream model rate cards. |
| **Active Virtual Keys** | Count | Number of unique client credentials actively dispatching traffic during the window. |
| **Classification Distribution** | Percentage (%) | Proportion of queries resolved to `low`, `medium`, and `high` tiers. |

---

## Date Range Controls & Presets

The analytics toolbar supports both rapid presets and custom date range filters:

- **Quick Presets**:
  - `Today`: Midnight UTC to the current timestamp.
  - `Last 7 Days`: Rolling seven-day interval.
  - `Last 30 Days`: Rolling thirty-day interval.
  - `All Time`: Full historical retention.
- **Custom UTC Ranges**: Choose explicit start and end calendar days. Timezone handling is normalized to UTC to guarantee consistent reporting across distributed teams.

---

## Tier Distribution & Cost Optimization

The tier distribution visualization demonstrates how effectively the automatic classification engine is deflecting traffic:

```
+---------------------------------------------------------------+
|  Tier Distribution Breakdown (Past 30 Days)                   |
|                                                               |
|  [==========================                  ]               |
|   LOW (58%)            MEDIUM (32%)   HIGH (10%)              |
+---------------------------------------------------------------+
```

- **High `low` Percentage**: Demonstrates significant cost savings by fulfilling routine greetings, extractions, and summaries on fast, economical models.
- **Targeted `high` Utilization**: Confirms that expensive reasoning models (o1, Opus) are reserved strictly for high-value complex code, math, and architecture queries.

---

## Team & Key Attribution

Administrators can drill down into usage across distinct organizational dimensions:

1. **Spend by Team**: Identifies budget burn rate per business unit or engineering department.
2. **Top Virtual Keys**: Pinpoints individual services, batch pipelines, or development keys with the highest token velocity.
3. **Model Provider Split**: Breaks down spend across underlying vendors (Azure OpenAI vs. Anthropic vs. Google Cloud vs. OpenAI Direct).

---

## Data Export & Integration

- **CSV / JSON Export**: Export detailed usage summaries directly from the admin console.
- **Prometheus & OpenTelemetry**: Flit can export metrics to standard Prometheus endpoints (`/metrics`) for ingestion into Grafana, Datadog, or enterprise APMs.

---

## Next Steps

- Integrate external tool servers in [MCP_TOOLS_AND_EXTERNAL_AGENTS.md](MCP_TOOLS_AND_EXTERNAL_AGENTS.md).
- Review client SDK examples and API specs in [GATEWAY_API_REFERENCE.md](GATEWAY_API_REFERENCE.md).
