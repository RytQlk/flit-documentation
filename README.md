# Flit documentation

> **Beta:** Flit is available to invited testers for evaluation. Product
> behavior and documentation may change as testing progresses.

Flit is an enterprise AI gateway and administration experience built around
LiteLLM. It gives teams a stable way to configure model deployments, issue
access-controlled virtual keys, understand usage, and route requests through
cost- and capability-oriented model tiers.

Flit is intended for teams that want one controlled entry point for multiple
LLM providers without exposing provider credentials or deployment details to
every application and user.

## What Flit provides

- A browser-based administration and chat experience.
- Local administrator accounts and optional Microsoft Entra sign-in.
- Virtual model tiers named `low`, `medium`, and `high`.
- The `flit` automatic selector for policy-aware request classification.
- Exact model deployment access when deliberate pinning is required.
- Virtual keys, teams, access controls, and usage analytics.
- LiteLLM-backed retry, failover, accounting, caching, and supported
  guardrails.
- Docker-based installation with bundled PostgreSQL and Redis for evaluation.

The beta does not arrive with model-provider credentials or preconfigured
deployments. An administrator must add at least one supported provider/model
deployment before inference requests can succeed.

## Current release

The current tester release is [`0.1.0-beta.3`](https://github.com/RytQlk/flit-documentation/releases/tag/0.1.0-beta.3).
Its application images are published in the public Flit Azure Container
Registry:

```text
acrflitpublic.azurecr.io/flit/flit-gateway:0.1.0-beta.3
acrflitpublic.azurecr.io/flit/flit-web:0.1.0-beta.3
```

Installations use readable version-tagged, immutable digest references from the
release manifest rather than relying directly on mutable tags. Gateway and Web
support both `linux/amd64` and `linux/arm64`. The images can be pulled
anonymously; testers do not need an Azure account or registry credentials.

Download the ZIP, manifest, and checksum file from the
[`0.1.0-beta.3` GitHub prerelease](https://github.com/RytQlk/flit-documentation/releases/tag/0.1.0-beta.3).

## Install Flit

Start with the [installation guide](documentation/INSTALLATION.md). It covers:

- macOS, Linux, and Windows prerequisites;
- downloading and inspecting a release;
- generating local secrets;
- pulling the public images;
- applying migrations in the required order;
- starting and verifying Flit;
- first login and smoke testing; and
- stopping, restarting, troubleshooting, and removing a test installation.

The installation does not require the private Flit source repository, Git,
Node.js, or a local compiler.

## Beta expectations

This release is intended for evaluation and structured testing, not an
unreviewed internet-facing production deployment.

- Use synthetic or approved test data.
- Do not expose the default HTTP endpoint directly to the internet.
- Keep generated secrets, virtual keys, provider credentials, and `.env` files
  out of issues and chat messages.
- Back up PostgreSQL and secrets before testing upgrades.
- Expect configuration and operational guidance to evolve during the beta.

## Documentation

| Document | Status | Purpose |
| --- | --- | --- |
| [Welcome & architecture](documentation/WELCOME_AND_OVERVIEW.md) | Available | Platform architecture, core pillars, and proxy request flow. |
| [Installation](documentation/INSTALLATION.md) | Available | Install and validate a published Docker release. |
| [Getting started](documentation/GETTING_STARTED.md) | WIP | First login, model setup, and first request. |
| [Configuration](documentation/CONFIGURATION.md) | WIP | Runtime options, identity, storage, and integrations. |
| [Troubleshooting](documentation/TROUBLESHOOTING.md) | WIP | Symptoms, diagnostics, and recovery procedures. |
| [Releases](documentation/RELEASES.md) | WIP | Release channels, compatibility, upgrade, and rollback policy. |
| [Security](SECURITY.md) | WIP | Private vulnerability reporting and safe diagnostics. |
| [Support](SUPPORT.md) | WIP | Beta support scope and bug-report expectations. |
| [Signing in & authentication](documentation/SIGNING_IN_AND_AUTHENTICATION.md) | Available | Local credentials, session security, and authentication governance. |
| [Microsoft SSO login](documentation/MICROSOFT_SSO_SETUP.md) | Available | Configure Microsoft Entra sign-in for Flit. |
| [Chat & document analysis](documentation/CHAT_AND_DOCUMENT_ATTACHMENTS.md) | Available | Web chat UI, model picker, file attachments, and thread management. |
| [Model administration & tiers](documentation/MODEL_ADMINISTRATION_AND_TIERS.md) | Available | Register upstream deployments, credentials, capability tiers, and failover. |
| [Automatic request classification](documentation/AUTOMATIC_REQUEST_CLASSIFICATION.md) | Available | Dynamic model tier routing using regex rules, classifier LLMs, or hybrid pipelines. |
| [Virtual keys, users & teams](documentation/VIRTUAL_KEYS_USERS_AND_TEAMS.md) | Available | Multi-tenant governance, virtual keys, team budgets, and rate limits. |
| [Guardrails & data protection](documentation/GUARDRAILS_AND_DATA_PROTECTION.md) | Available | Presidio PII detection, DLP redaction rules, and content safety filters. |
| [Usage analytics & cost observability](documentation/USAGE_ANALYTICS_AND_COST_OBSERVABILITY.md) | Available | Token consumption, latency metrics, spend tracking, and audit exports. |
| [MCP tools & external agents](documentation/MCP_TOOLS_AND_EXTERNAL_AGENTS.md) | Available | Model Context Protocol (MCP) tool integration and agent-to-agent workflows. |
| [Gateway API reference](documentation/GATEWAY_API_REFERENCE.md) | Available | OpenAI-compatible endpoints, authentication, model routing, and code examples. |

## Feedback

Invited testers should use the support channel supplied by the Flit team. When
reporting a problem, include the Flit version, operating system, Docker version,
the failing step, and sanitized logs. Never include passwords, tokens, provider
keys, session cookies, `.env` contents, or files from the `secrets/` directory.

## License

Licensing information will be added before the repository and release are
opened for general public use. No license is granted merely by the presence of
source or documentation in this repository.
