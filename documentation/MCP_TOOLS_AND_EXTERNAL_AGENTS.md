# 09 — MCP Tools & External Agents

Flit integrates with the open **Model Context Protocol (MCP)** and supports **Agent-to-Agent (A2A)** workflows, allowing models to securely connect to external tools, databases, and enterprise runtimes.

---

## Overview of Model Context Protocol (MCP)

The Model Context Protocol (MCP) is an open standard that allows LLM applications to discover, query, and execute tools running across distributed environments without requiring custom hard-coded integrations for every model provider.

```
+------------------+         MCP (SSE / HTTP)        +-------------------+
|                  |  ---------------------------->  |   MCP Server A    |
|   Flit Gateway   |  1. Tool & Resource Discovery   | (PostgreSQL Tool) |
|  Proxy & Agents  |  <----------------------------  +-------------------+
|                  |
|                  |  ---------------------------->  +-------------------+
|                  |  2. Authorized Execution Call   |   MCP Server B    |
|                  |  <----------------------------  | (Enterprise CRM)  |
+------------------+                                 +-------------------+
```

---

## Capabilities & Governance

- **Standardized Transports**: Connects to remote or local MCP servers over Server-Sent Events (SSE) or HTTP transports.
- **Dynamic Tool Schema Discovery**: Flit queries connected MCP servers on startup and periodically refreshes JSON Schema tool definitions.
- **Access Delegation**: Control which Teams and Virtual Keys are permitted to invoke specific MCP tools.
- **Audit Logging**: Every tool execution is recorded with execution duration, initiating identity, and return status code.

---

## Step-by-Step: Registering an MCP Server

1. Navigate to **Models & providers > MCP Servers** in the admin sidebar.
2. Click **Connect MCP server**.
3. Fill in connection parameters:
   - **Server Name**: e.g., `Corporate-Database-Tools`.
   - **Transport Type**: Select `Server-Sent Events (SSE)` or `HTTP POST`.
   - **Endpoint URL**: e.g., `https://mcp-internal.internal.corp/events`.
   - **Authentication**: Add optional bearer tokens, API keys, or custom headers.
4. Click **Connect & Probe**. Flit verifies connectivity, inspects available tools, and marks the server status as healthy.

---

## External Agent-to-Agent (A2A) Integrations

The **Agents** workspace allows administrators to register and govern external autonomous agent runtimes (e.g., CrewAI, AutoGen, LangGraph, or custom agent endpoints):

- **Endpoint Registration**: Register agent URLs and associate input/output schemas.
- **Interactive Sandbox**: Send sample payloads to test agent behavior directly inside the admin console.
- **Virtual Key Scoping**: Issue virtual keys specifically restricted to agent dispatch workflows.

---

## Security Boundaries & Execution Policies

1. **Network Isolation**: MCP servers running in untrusted environments should be deployed on isolated subnets with TLS termination.
2. **Credential Masking**: Secret headers passed to MCP servers are stored in the encrypted credential vault and are write-only.
3. **Execution Timeouts**: Enforce maximum execution timeouts (default 30 seconds) to prevent frozen agent worker threads.

---

## Next Steps

- Consult the complete API specification and cURL examples in [GATEWAY_API_REFERENCE.md](GATEWAY_API_REFERENCE.md).
- Return to the table of contents in [README.md](README.md).
