# 04 — Model Administration & Tiers

The **Models** workspace allows administrators to register upstream LLM deployments, connect provider credentials, assign models to public capability tiers, and define fallback mechanisms.

---

## Architectural Principles

1. **Immutable Identifiers**: Deployment ID, provider family, and provider-model ID cannot be edited once saved. This guarantees audit integrity and prevents broken routing pointers in active sessions.
2. **Write-Only Credentials**: API keys and connection tokens are securely transmitted to the encrypted credential store and are **never** returned in API read responses or browser DOMs.
3. **Strict Tier Membership**: Each registered text deployment belongs to exactly one public tier (`low`, `medium`, or `high`).
4. **Load Balancing & Failover**: Multiple deployments registered within the same tier automatically load-balance requests. If an upstream provider rate-limits or returns a 5xx error, Flit automatically falls back to secondary replicas in the group.

---

## Supported Providers & Protocols

Flit natively integrates with all major AI model providers:

| Provider | Supported Models | Required Credentials & Configuration |
| :--- | :--- | :--- |
| **OpenAI** | GPT-4o, GPT-4o-mini, o1, o3-mini | API Key, optional Organization ID |
| **Anthropic** | Claude 3.5 Sonnet, Claude 3.5 Haiku, Claude 3 Opus | API Key |
| **Azure OpenAI** | GPT-4o, GPT-4o-mini, Embeddings | Endpoint URL, API Key, Deployment Name, API Version |
| **Google Cloud Vertex / Gemini** | Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini 2.0 | Service Account Key JSON or Gemini API Key, Project ID, Location |
| **AWS Bedrock** | Claude 3.5 Sonnet, Llama 3.3, Amazon Titan | AWS Access Key ID, Secret Access Key, Region |
| **Ollama / Local Endpoints** | Llama 3, Mistral, Qwen, DeepSeek | Host URL (e.g., `http://host.docker.internal:11434`), Model Tag |
| **Custom OpenAI-Compatible** | Any vLLM, TGI, or LiteLLM proxy instance | Base URL, API Key, Model Identifier |

---

## Step-by-Step: Adding an Upstream Deployment

1. Navigate to **Models & providers** in the admin console sidebar (`/admin`).
2. Click **Add deployment** in the top action bar.
3. **General Information**:
   - **Deployment ID**: Enter a unique slug (e.g., `azure-gpt4o-eastus-01`).
   - **Display Name**: Friendly name for UI lists (e.g., `Azure GPT-4o (East US)`).
   - **Provider Family**: Select `Azure OpenAI`, `OpenAI`, `Anthropic`, etc.
   - **Provider Model ID**: The upstream model name as expected by the provider (e.g., `gpt-4o`).
4. **Tier Assignment**:
   - Assign the deployment to `low`, `medium`, or `high`.
5. **Connection Credentials**:
   - Enter the API key, endpoint URL, or required headers.
6. Click **Save deployment**. Flit validates connectivity and commits the deployment. Traffic routed to that tier will immediately begin utilizing the new deployment.

---

## Fallback & High-Availability Configurations

When configuring production deployments:
- Register at least two independent provider deployments for each tier (e.g., Azure OpenAI as primary, Anthropic or OpenAI Direct as secondary).
- Set up health probes (`/health/ready`) to ensure unhealthy provider endpoints are removed from rotation before client requests fail.

---

## Next Steps

- Proceed to [AUTOMATIC_REQUEST_CLASSIFICATION.md](AUTOMATIC_REQUEST_CLASSIFICATION.md) to configure how incoming prompts are categorized into these tiers.
- Review [VIRTUAL_KEYS_USERS_AND_TEAMS.md](VIRTUAL_KEYS_USERS_AND_TEAMS.md) to manage virtual keys that grant access to these models.
