# 02 — Signing In & Authentication Governance

Flit provides enterprise-grade identity and session security with support for local administrative credentials and Microsoft Entra ID (Single Sign-On).

---

## Authentication Modes

### 1. Local Administrator Authentication
- **Initial Setup**: Built-in bootstrap account with password-based authentication.
- **Mandatory Rotation**: Initial temporary passwords require mandatory rotation upon first login.
- **Complexity Requirements**: Passwords must be at least 12 characters long and include numbers and special characters.
- **Storage**: Credentials are securely salted and hashed using bcrypt before database persistence.

### 2. Microsoft Entra ID (Single Sign-On)
- **Protocols**: Enterprise single sign-on integration via OAuth 2.0 / OpenID Connect (OIDC).
- **Tenant Isolation**: When configured with your organization's Azure tenant (`AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, and `AZURE_CLIENT_SECRET`), users authenticate directly with corporate credentials.
- **Just-In-Time Provisioning**: Authenticated corporate identities are automatically provisioned in Flit with appropriate role attribution (`admin` or `user`).

---

## Brute-Force & Lockout Protection

To safeguard gateway administration endpoints against credential stuffing and brute-force attacks:

- **Lockout Threshold**: 5 consecutive failed login attempts.
- **Lockout Duration**: 15 minutes before further attempts are evaluated.
- **Session Security**: Sessions are secured using encrypted, HTTP-only, SameSite cookies with CSRF token protection on state-modifying requests.
- **Header Hardening**: Emits strict security headers including `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and Content Security Policy (CSP).

---

## Role-Based Access Control (RBAC)

Flit separates access into two primary permission profiles:

| Role | Default Landing Page | Permissions |
| :--- | :--- | :--- |
| **`admin`** | `/admin` | Manage models, providers, classifiers, virtual keys, teams, guardrails, analytics, and system configurations. |
| **`user`** | `/chat` | Access web chat, attach documents for analysis, manage personal chat history, and toggle themes. Cannot access admin configurations. |

---

## Session Lifecycle & Expiration

- **Inactivity Timeout**: Configurable session lifetime (defaults to 8 hours).
- **Explicit Logout**: Clicking **Sign Out** immediately invalidates the server-side session token and clears client cookies.
- **Password Reset**: Changing account credentials revokes all existing active sessions for that identity.

---

## Next Steps

- Review [CHAT_AND_DOCUMENT_ATTACHMENTS.md](CHAT_AND_DOCUMENT_ATTACHMENTS.md) for end-user chat features.
- Jump to [VIRTUAL_KEYS_USERS_AND_TEAMS.md](VIRTUAL_KEYS_USERS_AND_TEAMS.md) for administering user accounts, teams, and virtual keys.
