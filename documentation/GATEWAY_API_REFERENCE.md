# 10 — OpenAI-Compatible Gateway API Reference

Flit provides full protocol compatibility with standard OpenAI client libraries, HTTP clients, and AI frameworks (e.g., LangChain, LlamaIndex, LiteLLM SDK). Applications can point to Flit simply by updating the API base URL and passing a Flit Virtual Key.

---

## Base URLs & Authentication

- **Gateway Base URL**: `http://<flit-host>:4000/v1` (or your configured custom gateway domain).
- **Authentication**: Pass your Flit Virtual Key in the standard HTTP `Authorization` header:

```http
Authorization: Bearer fk-your-virtual-key-secret
```

---

## Primary Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/v1/chat/completions` | Standard streaming and non-streaming chat completions. |
| `POST` | `/v1/responses` | Specialized responses endpoint supporting document and tabular attachments. |
| `GET` | `/v1/models` | Lists authorized model tiers and pinned deployments accessible to the caller's key. |
| `GET` | `/health/ready` | Gateway, cache, and database readiness probe. |

---

## Model Identifiers

When sending requests to `/v1/chat/completions`, the `"model"` field accepts the following formats:

- `"model": "auto"` *(Recommended)*: Dynamically routed via the active classification engine (`Regex` -> `Classifier LLM`) to `low`, `medium`, or `high`.
- `"model": "low"`: Explicitly routes to the configured Low tier pool (fast, economical).
- `"model": "medium"`: Explicitly routes to the configured Medium tier pool (balanced).
- `"model": "high"`: Explicitly routes to the configured High tier pool (reasoning).
- `"model": "<exact-deployment-id>"`: Direct invocation of an upstream deployment (e.g., `"azure-gpt4o-eastus-01"`), if permitted by the virtual key's model whitelist.

---

## Code Examples

### 1. cURL Example — Non-Streaming Chat Completion

```bash
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer fk-your-virtual-key-secret" \
  -d '{
    "model": "auto",
    "messages": [
      {
        "role": "system",
        "content": "You are a concise enterprise assistant."
      },
      {
        "role": "user",
        "content": "Compare synchronous vs asynchronous message queues."
      }
    ],
    "temperature": 0.7,
    "max_tokens": 500
  }'
```

#### Sample Response Payload:
```json
{
  "id": "chatcmpl-9b8c2dfa",
  "object": "chat.completion",
  "created": 1728835200,
  "model": "flit-tier-medium",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Synchronous message queues require the sender to wait for acknowledgment..."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 28,
    "completion_tokens": 142,
    "total_tokens": 170
  }
}
```

---

### 2. cURL Example — Streaming Chat Completion

```bash
curl -N -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer fk-your-virtual-key-secret" \
  -d '{
    "model": "auto",
    "stream": true,
    "messages": [
      {
        "role": "user",
        "content": "Write a Python script to parse a CSV file and output JSON."
      }
    ]
  }'
```

---

### 3. Python Example — Official `openai` SDK

```python
from openai import OpenAI

# Initialize client pointing to Flit gateway
client = OpenAI(
    base_url="http://localhost:4000/v1",
    api_key="fk-your-virtual-key-secret",
)

# Dispatch completion with automatic routing
response = client.chat.completions.create(
    model="auto",
    messages=[
        {"role": "system", "content": "You are an enterprise AI assistant."},
        {"role": "user", "content": "Summarize the key requirements for SOC2 compliance."}
    ],
    temperature=0.5,
)

print(response.choices[0].message.content)
print(f"\nTokens Consumed: {response.usage.total_tokens}")
```

---

### 4. TypeScript / Node.js Example

```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  baseURL: "http://localhost:4000/v1",
  apiKey: process.env.FLIT_VIRTUAL_KEY || "fk-your-virtual-key-secret",
});

async function main() {
  const completion = await openai.chat.completions.create({
    model: "auto",
    messages: [
      { role: "user", content: "Draft a release announcement for version 2.4." }
    ],
  });

  console.log(completion.choices[0].message.content);
}

main().catch(console.error);
```

---

## Standard Error Codes & Responses

Flit emits RFC 7807 compliant error payloads:

| HTTP Status | Error Code | Reason & Remediation |
| :--- | :--- | :--- |
| **`401 Unauthorized`** | `invalid_api_key` | Virtual key missing, malformed, or expired. |
| **`403 Forbidden`** | `model_not_allowed` | The virtual key is not authorized for the requested model tier or deployment. |
| **`429 Too Many Requests`** | `rate_limit_exceeded` | The virtual key has exceeded its configured RPM or TPM threshold. |
| **`429 Too Many Requests`** | `budget_exceeded` | The monthly dollar spend limit for the virtual key or owning team has been reached. |
| **`400 Bad Request`** | `guardrail_violation` | Sensitive PII or blocked content was detected under a `BLOCK` guardrail policy. |
| **`502 Bad Gateway`** | `upstream_provider_error` | Upstream model provider returned an error, and all configured fallbacks failed. |

---

## Next Steps

- Return to the [Table of Contents](README.md) for navigation across all sections.
