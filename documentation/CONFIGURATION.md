# Configure a Flit Docker release

This guide describes the supported, non-secret settings in
`src/deployment/docker/.env` in a standard Flit release. Start with the file
included in the same release ZIP:

```bash
cp src/deployment/docker/.env.example src/deployment/docker/.env
python3 scripts/setup_runtime_secrets.py
```

Do not copy an `.env` from an older Flit or Aquila release. Configuration may
change between beta versions. Keep passwords, keys, and client secrets out of
`.env`; Flit stores them as individual files under `SECRETS_DIR`.

## How Docker Compose reads the configuration

Run Docker Compose commands from `src/deployment/docker`. Docker Compose reads
the `.env` in that directory automatically.

The setup script also writes `COMPOSE_FILE` to `.env`. This records installation
choices that require additional Compose files, including immutable production
images, external PostgreSQL, Microsoft Entra sign-in, and Flit-managed Presidio.

You normally do not need to edit `COMPOSE_FILE`. Rerun setup when changing one
of those choices, then inspect the result:

```bash
cd src/deployment/docker
docker compose config --quiet
docker compose config --images
```

## Release-owned settings

These values identify the release and should not be edited.

| Setting | Purpose |
| --- | --- |
| `FLIT_RELEASE_DISTRIBUTION` | Marks a published distribution. Setup uses it to select the production-image overlay. It must be `true`. |
| `WEB_IMAGE` | Exact version-tagged, digest-qualified Web image used by the runtime and Flit migration job. |
| `GATEWAY_IMAGE` | Exact version-tagged, digest-qualified Gateway image used by the runtime and LiteLLM migration job. |
| `PRESIDIO_IMAGE_TAG` | Readable version of the optional managed Presidio images. |
| `PRESIDIO_ANALYZER_IMAGE_DIGEST` | Immutable Presidio Analyzer image digest. |
| `PRESIDIO_ANONYMIZER_IMAGE_DIGEST` | Immutable Presidio Anonymizer image digest. |

Do not replace a digest with `latest`, a development tag, or another image. The
application images must match `release-manifest.json`.

## PostgreSQL

The setup script asks whether to use bundled or external PostgreSQL and updates
the relevant values.

| Setting | Default | Purpose |
| --- | --- | --- |
| `POSTGRES_HOST` | `db` | Host used by the bundled deployment. `db` is the private Compose service name. |
| `POSTGRES_PORT` | `5432` | PostgreSQL TCP port. |
| `POSTGRES_USER` | `flit` | Database login and bundled-database initialization user. |
| `POSTGRES_DB` | `flit` | Database used by Flit. |
| `POSTGRES_SSLMODE` | empty | PostgreSQL TLS policy. Leave empty for bundled PostgreSQL; use the mode required by an external database. |
| `POSTGRES_VOLUME_NAME` | `flit_pgdata` | Docker named volume holding bundled PostgreSQL data. |
| `EXTERNAL_POSTGRES_HOST` | empty | Required external PostgreSQL hostname when that setup option is selected. |
| `EXTERNAL_POSTGRES_PORT` | `5432` | External PostgreSQL port. |

The database password is stored in `SECRETS_DIR/postgres_password`, not `.env`.
Changing a volume name selects a different volume; it does not rename or
migrate existing data.

## Redis

Redis is bundled in the current beta and authenticated using a secret file.

| Setting | Default | Purpose |
| --- | --- | --- |
| `REDIS_HOST` | `redis` | Redis host; `redis` is the bundled Compose service name. |
| `REDIS_PORT` | `6379` | Redis TCP port. |
| `REDIS_DATABASE` | `0` | Logical Redis database number. |
| `REDIS_SCHEME` | `redis` | Connection scheme. `rediss` is for a TLS-enabled external service when supported by the deployment. |
| `REDIS_VOLUME_NAME` | `flit_redisdata` | Docker named volume holding bundled Redis data. |
| `WEB_REDIS_CONNECT_TIMEOUT_SECONDS` | `2` | Maximum time for a Web worker to establish a Redis connection. Must be positive. |
| `WEB_REDIS_COMMAND_TIMEOUT_SECONDS` | `2` | Maximum time for a Web Redis operation. Must be positive. |
| `WEB_REDIS_MAX_CONNECTIONS` | `5` | Maximum Redis connections per Web worker. |

The Redis password is stored in `SECRETS_DIR/redis_password`. Total possible Web
connections increase with both `WEB_WORKERS` and
`WEB_REDIS_MAX_CONNECTIONS`.

## Classifier refresh

| Setting | Default | Purpose |
| --- | --- | --- |
| `FLIT_CLASSIFIER_REFRESH_INTERVAL_SECONDS` | `30` | How often each Gateway worker checks PostgreSQL for classifier changes. Values are bounded internally to 1–300 seconds. |

Classification requests use worker memory rather than querying PostgreSQL in
the request path. A temporary refresh failure retains the previous valid state.

## Microsoft Entra sign-in and browser sessions

Use `scripts/setup_runtime_secrets.py` to configure Entra sign-in. For the full
registration workflow, see [Microsoft SSO login setup](MICROSOFT_SSO_SETUP.md).

| Setting | Default | Purpose |
| --- | --- | --- |
| `MICROSOFT_TENANT_ID` | empty | Microsoft Entra directory/tenant allowed to sign in. |
| `MICROSOFT_CLIENT_ID` | empty | Application/client ID and expected token audience. |
| `MICROSOFT_REDIRECT_URI` | `http://localhost:4000/auth/sso/callback` | OAuth callback. It must exactly match the URI registered in Entra. |
| `MICROSOFT_POST_LOGIN_REDIRECT_URL` | `http://localhost:4000/chat` | Browser destination after successful sign-in. |
| `MICROSOFT_TOKEN_REFRESH_BUFFER_SECONDS` | `120` | Refresh delegated Microsoft access before expiry by this many seconds. |
| `SESSION_TTL_SECONDS` | `3600` | Browser-session lifetime. Flit constrains it to 300–86,400 seconds. |
| `SSO_COOKIE_SECURE` | `false` | Whether the browser session cookie requires HTTPS. Use `true` for HTTPS and `false` only for local HTTP testing. |

The Microsoft client secret is stored in
`SECRETS_DIR/microsoft_client_secret`. Do not put it in `.env`.

## Azure AI Search retrieval

Flit can enrich an authenticated chat request with results from an Azure AI
Search index. It uses the signed-in user's delegated Microsoft identity rather
than an Azure Search API key.

| Setting | Default | Purpose |
| --- | --- | --- |
| `RAG_ENABLED` | `false` | Enables Azure AI Search retrieval. Leave disabled until Entra and the search index are ready. |
| `AZURE_SEARCH_ENDPOINT` | empty | Search service endpoint, such as `https://example.search.windows.net`. Required when enabled. |
| `AZURE_SEARCH_INDEX_NAME` | empty | Index containing searchable document chunks. Required when enabled. |
| `AZURE_SEARCH_API_VERSION` | `2026-05-01-preview` | Azure AI Search API version used by this release. |
| `RAG_MIN_SCORE` | `0.5` | Minimum non-negative relevance score accepted from retrieval. |
| `RAG_TOP_K` | `5` | Maximum number of search matches considered for context. Must be positive. |
| `RAG_TIMEOUT_SECONDS` | `15` | Timeout for a retrieval attempt. Must be positive. |
| `RAG_RETRY_ATTEMPTS` | `4` | Number of attempts allowed for retryable retrieval failures. Must be positive. |
| `RAG_FAILURE_MODE` | `bypass` | `bypass` continues without context after failure; `fail` rejects the request. |

When `RAG_ENABLED=true`, the tenant ID, client ID, endpoint, and index name must
all be configured. Confirm the required delegated permission in the
[Microsoft SSO guide](MICROSOFT_SSO_SETUP.md) before enabling retrieval.

## Web and Gateway runtime sizing

| Setting | Default | Purpose |
| --- | --- | --- |
| `WEB_WORKERS` | `2` | Number of Web server processes. |
| `WEB_DB_POOL_MIN_SIZE` | `1` | Minimum PostgreSQL connections per Web worker. Must be at least 1. |
| `WEB_DB_POOL_MAX_SIZE` | `5` | Maximum PostgreSQL connections per Web worker. Must be no smaller than the minimum. |
| `WEB_FORWARDED_ALLOW_IPS` | `*` | Proxies whose forwarded headers the Web server trusts. Keep Web private behind the supplied Nginx boundary. |
| `GATEWAY_WORKERS` | setup-selected | Number of Gateway worker processes. Setup recommends Docker's available CPU count, bounded to 1–8, and lets the operator override it. |

Increasing worker counts increases memory use and aggregate database/cache
connections. Maximum Web PostgreSQL allocation is approximately
`WEB_WORKERS × WEB_DB_POOL_MAX_SIZE`.

`WEB_WORKERS` remains fixed at two unless an operator deliberately edits the
configuration. On first setup, `GATEWAY_WORKERS` is suggested from
`docker info`; host CPU detection is used only when Docker cannot report its
allocation. Rerunning setup preserves the existing positive Gateway value as
the prompt default rather than silently resizing the installation.

## Custom model costs

| Setting | Default | Purpose |
| --- | --- | --- |
| `CUSTOM_MODEL_COST_MAP_URL` | empty | Optional URL containing a LiteLLM-compatible JSON model-cost map loaded when Gateway workers start. |

Use a custom map only when a model is not represented correctly by the catalog
shipped in the release. The URL must be reachable by every Gateway worker
during startup. Invalid or unavailable configured data fails registration
rather than silently using incorrect costs.

For an operator-owned JSON file instead of a URL, use the packaged
`overlays/docker-compose.custom-costs.yml` and
`custom-model-costs.example.json`. Set `CUSTOM_MODEL_COST_MAP_FILE` to the host
file path when invoking that overlay. URL and file inputs are mutually exclusive.

## Local authentication

| Setting | Default | Purpose |
| --- | --- | --- |
| `FLIT_BOOTSTRAP_ADMIN_USERNAME` | `admin` | Username created only when migrations initialize an empty Flit schema. Changing it later does not rename an account. |
| `FLIT_MINIMUM_PASSWORD_LENGTH` | `12` | Minimum length for newly accepted local passwords. Must be positive. |
| `FLIT_LOGIN_LOCKOUT_FAILURES` | `5` | Failed-login count that temporarily locks an account. Flit enforces a minimum of 3. |
| `FLIT_LOGIN_LOCKOUT_SECONDS` | `900` | Temporary lock duration. Flit enforces a minimum of 60 seconds. |
| `FLIT_ENTRA_ADMIN_IDENTITIES` | empty | Preferred comma-separated Entra administrator bindings in `<tenant-id>:<object-id>` form. |
| `FLIT_ENTRA_ADMIN_EMAILS` | empty | Optional first-login administrator bootstrap bindings by email. Stable identity bindings are preferred. |

The bootstrap password is stored in
`SECRETS_DIR/flit_bootstrap_admin_password` and is consumed only when an empty
schema is initialized.

## Nginx TLS

| Setting | Default | Purpose |
| --- | --- | --- |
| `NGINX_TLS_DIR` | `./tls` | Directory containing `tls.crt` and `tls.key` for the packaged TLS overlay. |

The TLS overlay publishes ports 80 and 443, redirects HTTP to HTTPS, and mounts
the certificate directory read-only. Flit does not request, renew, or rotate
certificates.

To activate it, preserve the setup-generated files in `COMPOSE_FILE`, append
`docker-compose.tls.yml`, and enable secure cookies. macOS or Linux example:

```dotenv
COMPOSE_FILE=docker-compose.yml:overlays/docker-compose.production-images.yml:docker-compose.tls.yml
NGINX_TLS_DIR=./tls
SSO_COOKIE_SECURE=true
```

Windows uses a semicolon instead of a colon between Compose files.

## Secret directory

| Setting | Default | Purpose |
| --- | --- | --- |
| `SECRETS_DIR` | `./secrets` | Directory containing individual runtime credential files mounted as Docker secrets. |

Depending on selected features, it contains:

```text
postgres_password
redis_password
litellm_master_key
litellm_salt_key
flit_cookie_encryption_key
flit_bootstrap_admin_password
microsoft_client_secret
```

The Microsoft file is present only when Entra sign-in is configured. Keep this
directory owner-readable, exclude it from source control, and back it up only
through an approved secret-management process. Rerunning setup preserves
existing secret files and does not silently rotate credentials.

## Validate a change

After editing non-secret configuration, validate before restarting services:

```bash
python3 scripts/setup_runtime_secrets.py --validate-only
cd src/deployment/docker
docker compose config --quiet
docker compose config --images
```

Review the rendered image references and selected services. Apply runtime-only
changes by recreating affected containers. Run migrations only when the
matching release's installation or upgrade instructions require them.
