# 06 — Virtual Keys, Users & Teams Access Governance

Flit enforces multi-tenant enterprise governance, quota controls, and access boundaries through **Virtual Keys** and **Team** abstractions.

---

## Architecture of Multi-Tenancy

```
                              +-----------------------+
                              |      Organization     |
                              +-----------+-----------+
                                          |
                      +-------------------+-------------------+
                      |                                       |
                      v                                       v
             +-----------------+                     +-----------------+
             |   Team: Alpha   |                     |    Team: Beta   |
             | Monthly Cap: $1k|                     | Monthly Cap: $5k|
             +--------+--------+                     +--------+--------+
                      |                                       |
           +----------+----------+                 +----------+----------+
           |                     |                 |                     |
           v                     v                 v                     v
    +-------------+       +-------------+   +-------------+       +-------------+
    | Virtual Key |       | Virtual Key |   | Virtual Key |       | Virtual Key |
    |   (Backend) |       |   (Mobile)  |   | (Analytics) |       |  (Internal) |
    | RPM: 100    |       | RPM: 50     |   | TPM: 500k   |       | Scoped: low |
    +-------------+       +-------------+   +-------------+       +-------------+
```

---

## 1. Virtual API Keys

Applications, services, and developers never receive direct provider credentials (such as raw OpenAI or Anthropic API keys). Instead, they consume Flit using **Virtual Keys** (`fk-...`):

### Key Capabilities & Policy Constraints
- **Model Whitelisting**: Restrict a virtual key to specific tiers (`auto`, `low`, `medium`, `high`) or individual pinned deployments.
- **Rate Limits**:
  - **Requests Per Minute (RPM)**: Throttles peak request concurrency to protect upstream quotas.
  - **Tokens Per Minute (TPM)**: Throttles token volume to prevent runaway agent loops.
- **Spend Budgets**: Define hard dollar spend caps per calendar month. Requests automatically return `HTTP 429 Insufficient Budget` when limits are reached.
- **Expiration Policies**: Issue short-lived keys for contractors, CI/CD pipelines, or temporary evaluation.
- **One-Time Secret Reveal**: The plaintext API key secret is presented exactly once upon creation. Flit stores only cryptographically secure hashes.

---

## 2. Teams & Project Ownership

Teams group users and virtual keys to represent business units, departments, or application projects:

- **Shared Budgets**: Set total monthly budgets allocated across all keys belonging to the team.
- **Default Tier Rules**: Configure team-level default model fallbacks.
- **Team Analytics**: Track token consumption, cost attribution, and request trends partitioned by team.
- **Member Management**: Assign users to teams with specific roles (`owner`, `member`, `viewer`).

---

## 3. User Identity & RBAC

Flit supports both internal directory accounts and enterprise federated single sign-on:

- **Local Users**: Created and managed by administrators with salted bcrypt password storage.
- **Microsoft Entra ID (SSO)**: Automated user mapping based on enterprise email domains and groups.
- **Role Permissions**:
  - `admin`: Full configuration access to gateway settings, providers, teams, keys, and security guardrails.
  - `user`: Access restricted to the web chat interface and document attachments.

---

## Managing Virtual Keys via Admin Console

1. Navigate to **Access governance > Virtual keys** in the admin sidebar.
2. Click **Create virtual key**.
3. Enter key details:
   - **Key Name / Description**: e.g., "Customer Portal Production".
   - **Team Assignment**: Select the owning team (e.g., "Frontend Apps").
   - **Allowed Models**: Select `auto`, or restrict to specific tiers/deployments.
   - **Budget Cap & Reset Interval**: e.g., `$500.00 / month`.
   - **Rate Limits**: e.g., `120 RPM`.
4. Click **Generate key**.
5. Copy the revealed key (`fk-...`) and store it securely.

---

## Next Steps

- Explore in-proxy DLP and privacy controls in [GUARDRAILS_AND_DATA_PROTECTION.md](GUARDRAILS_AND_DATA_PROTECTION.md).
- Learn how spend and tokens are tracked in [USAGE_ANALYTICS_AND_COST_OBSERVABILITY.md](USAGE_ANALYTICS_AND_COST_OBSERVABILITY.md).
