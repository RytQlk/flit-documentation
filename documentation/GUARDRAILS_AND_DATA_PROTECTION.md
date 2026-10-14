# 07 — Guardrails & Data Protection

Flit integrates Data Loss Prevention (DLP), Personally Identifiable Information (PII) detection, and content safety filters directly into the gateway proxy layer.

---

## Architectural Guardrail Pipeline

```
          Incoming Client Prompt
                     |
                     v
      +------------------------------+
      |  Presidio DLP Analyzer Layer |
      |   (Scans text before proxy)  |
      +--------------+---------------+
                     |
         Sensitive Entity Detected?
         +-----------+-----------+
         |                       |
        YES                      NO
         |                       |
         v                       v
    Action Policy?        Pass Unaltered Prompt
    +----+----+           to Downstream Model
    |         |
    v         v
 [REDACT]   [BLOCK]
    |         |
    |         +----> Return HTTP 400 Bad Request
    v                "Sensitive data detected"
 Replace with
 <EMAIL_ADDRESS>,
 <CREDIT_CARD>, etc.
    |
    v
 Send Sanitized Prompt
 to Downstream LLM Provider
```

---

## Microsoft Presidio Integration

Flit embeds Microsoft Presidio analyzers to detect and handle sensitive entities:

### Detected Entity Categories
- **Financial & Banking**: Credit card numbers, bank account numbers, IBANs, CVVs.
- **National Identifiers**: US Social Security Numbers (SSN), UK National Insurance numbers, national tax IDs.
- **Contact & Personal Data**: Email addresses, mobile phone numbers, physical addresses.
- **Technical Identifiers**: IP addresses, MAC addresses, internal hostnames.
- **Authentication Secrets**: Detected private keys, API credentials, passwords, JWT tokens.

---

## Action Policies

Administrators can configure granular action policies per rule or across the organization:

### 1. Redact (Masking) — Default
- Replaces detected sensitive text with typed entity tokens (e.g., `<EMAIL_ADDRESS>`, `<CREDIT_CARD_NUMBER>`, `<US_SSN>`).
- **Benefit**: Protects customer and employee privacy while allowing the LLM to complete analytical, drafting, or coding tasks with full surrounding context.
- Sanitization occurs completely before the prompt leaves the organization's boundary.

### 2. Block (Strict Rejection)
- Immediately terminates the request before sending data to upstream LLM providers.
- Emits an `HTTP 400 Bad Request` with structured error details specifying the violation category without echoing the secret.

---

## Prompt Injection & Jailbreak Defense

In addition to PII redaction, Flit provides safety checks against adversarial prompt manipulation:
- **System Prompt Integrity**: Verifies that user turns do not attempt to override system guardrails or leak internal instructions.
- **Heuristic Pattern Analysis**: Blocks common jailbreak framing patterns and unauthorized instruction injection.

---

## Auditing, Compliance & Logging Safety

- **Zero-Storage of Raw Secrets**: Flit logs metadata regarding detection events (e.g., entity type, timestamp, virtual key ID, character offset) but **never** persists raw unredacted values in persistent disk logs or database rows.
- **Audit Reports**: Compliance officers can export redaction frequency reports and review trend lines by department or team in the admin console.

---

## Next Steps

- Explore usage metrics and token trends in [USAGE_ANALYTICS_AND_COST_OBSERVABILITY.md](USAGE_ANALYTICS_AND_COST_OBSERVABILITY.md).
- Connect external tool servers in [MCP_TOOLS_AND_EXTERNAL_AGENTS.md](MCP_TOOLS_AND_EXTERNAL_AGENTS.md).
