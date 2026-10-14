# 01 — Welcome & Architecture

**Flit** is an enterprise AI gateway and administration platform built to provide centralized token visibility, automatic model classification, intelligent routing, and enterprise-grade data protection.

---

## Core Value Pillars

1. **Token & Spend Visibility**: Direct, auditable insight into LLM consumption, token flow, and costs across teams and virtual keys.
2. **Automatic Model Classification**: Dynamic routing between **Low**, **Medium**, and **High** capability tiers using fast in-process Regex and Classifier LLMs to minimize inference spend without sacrificing output quality.
3. **Team Governance & Budgets**: Shared team limits, scoped virtual keys, and spend tracking with native LiteLLM accounting.
4. **Data Protection & Guardrails**: Content redaction and PII protection powered by Microsoft Presidio before prompts ever reach external providers.
5. **Unified Chat Experience**: Clean, modern web chat interface for end users with markdown formatting, file attachments, and model selection.

---

## Gateway Architecture & Flow

```
                      +---------------------------------------+
                      |       Clients & Applications          |
                      |  (Web Chat UI, SDKs, cURL, Scripts)   |
                      +-------------------+-------------------+
                                          |
                                          v  [HTTP / OpenAI-compatible API]
                      +-------------------+-------------------+
                      |             Flit Proxy                |
                      |                                       |
                      |  1. Virtual Key Auth & Rate Limiting  |
                      |  2. Presidio DLP & PII Redaction      |
                      |  3. Classification Engine             |
                      |     (Regex -> Classifier LLM)         |
                      |  4. Dynamic Tier Routing              |
                      +-------------------+-------------------+
                                          |
                    +---------------------+---------------------+
                    |                     |                     |
                    v                     v                     v
            +---------------+     +---------------+     +---------------+
            |   LOW Tier    |     |  MEDIUM Tier  |     |   HIGH Tier   |
            | (Fast/Cheap)  |     |   (Balanced)  |     |  (Reasoning)  |
            +---------------+     +---------------+     +---------------+
            | GPT-4o-mini   |     | GPT-4o        |     | o1 / o3-mini  |
            | Claude Haiku  |     | Claude Sonnet |     | Claude Opus   |
            | Gemini Flash  |     | Gemini Pro    |     | High-end LLMs |
            +---------------+     +---------------+     +---------------+
```

---

## The Three-Tier Model Hierarchy

Rather than exposing sprawling provider-specific identifiers to application developers, Flit normalizes text models into three stable tiers:

| Tier | Target Workloads | Typical Target Models | Latency & Cost Profile |
| :--- | :--- | :--- | :--- |
| **`low`** | High-throughput, simple extraction, summarizing, categorization, greetings | GPT-4o-mini, Claude 3.5 Haiku, Gemini Flash | Sub-second latency, lowest token cost (up to 95% savings) |
| **`medium`** | General conversational assistance, drafting, and structured analysis | GPT-4o, Claude 3.5 Sonnet, Gemini Pro | Balanced capability and cost |
| **`high`** | Complex multi-step reasoning, advanced code synthesis, mathematical proofs | o1, o3-mini, Claude Opus | Maximum capability for demanding tasks |

---

## Routing Principles

- **Automatic Route (`auto`)**: Applications sending requests with `"model": "auto"` allow Flit's classification engine to evaluate the incoming prompt and dynamically route to `low`, `medium`, or `high`.
- **Pinned Tiers**: Clients can explicitly pass `"model": "low"`, `"model": "medium"`, or `"model": "high"` to bypass classification when prompt requirements are already known.
- **Direct Deployments**: Specific upstream deployments (e.g., `openai-gpt4o-prod`) can be targeted directly when explicit pinning is permitted by the caller's Virtual Key.

---

## Next Steps

- Proceed to [SIGNING_IN_AND_AUTHENTICATION.md](SIGNING_IN_AND_AUTHENTICATION.md) to understand authentication and security settings.
- Explore [CHAT_AND_DOCUMENT_ATTACHMENTS.md](CHAT_AND_DOCUMENT_ATTACHMENTS.md) for end-user chat features.
- Review [MODEL_ADMINISTRATION_AND_TIERS.md](MODEL_ADMINISTRATION_AND_TIERS.md) to register model providers.
