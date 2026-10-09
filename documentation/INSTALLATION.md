# Install Flit `0.1.0-beta.2`

> **Work in progress:** These instructions are ready for review, but the public
> release asset has not been uploaded yet. Do not substitute files from an old
> Aquila deployment package.

This guide installs a published Flit release on macOS, Linux, a Linux virtual
machine, or Windows. It is written for invited beta testers and does not require
the Flit source repository, Git, Node.js, a compiler, Azure CLI, or Azure
credentials.

## What the release contains

The release ZIP contains Docker Compose definitions, configuration templates,
the interactive setup script, Nginx configuration, and a manifest with exact
image digests. Docker pulls the Web and Gateway images anonymously from:

```text
acrflitpublic.azurecr.io/flit/flit-web
acrflitpublic.azurecr.io/flit/flit-gateway
```

PostgreSQL, Redis, Nginx, and optional Presidio images come from their public
registries. The machine therefore needs outbound HTTPS access to Azure
Container Registry, Docker Hub, and GitHub Container Registry when Presidio is
enabled.

## Before you begin

For a comfortable evaluation environment, allocate at least:

- 4 CPU cores;
- 8 GB RAM; and
- 20 GB of free Docker disk space.

More space may be needed when testing multiple releases or retaining database
backups. Port `4000` must be available on the installation host.

Choose the database mode before setup:

- **Bundled PostgreSQL** is recommended for local evaluation. Its data is kept
  in a Docker-managed volume named `flit_pgdata` by default.
- **External PostgreSQL** is for environments that already have an approved
  database, backup process, network path, and operational owner.

Redis is bundled in this beta and uses `flit_redisdata` by default. Optional
Microsoft Entra sign-in requires an application registration. Optional
Presidio adds two more containers and consumes additional memory.

The default installation uses HTTP. Do not expose it directly to the internet.
Use a trusted TLS ingress or the supplied TLS overlay with properly managed
certificates for any remote environment.

## 1. Install the prerequisites

### macOS

Install Python 3 and Docker Desktop with Docker Compose v2. Start Docker Desktop
and confirm:

```bash
python3 --version
docker version
docker compose version
```

### Linux or a Linux VM

Install Python 3, Docker Engine, and the Docker Compose v2 plugin through your
distribution's supported package mechanism. Confirm:

```bash
python3 --version
docker version
docker compose version
```

Run Docker consistently as root or as a user authorized to access the Docker
socket. Docker access is effectively root-level access to the host. If this is
a VM, allow inbound traffic only through the intended ingress.

### Windows

Install Python 3 and Docker Desktop using Linux containers and the WSL2 backend.
Use either PowerShell or Git Bash and confirm:

```powershell
python --version
docker version
docker compose version
```

If `python` is unavailable, use `py -3` in place of `python3` in this guide.

## 2. Download the release

Download these assets from the
[Flit documentation releases](https://github.com/RytQlk/flit-documentation/releases)
page:

```text
flit-0.1.0-beta.2.zip
flit-0.1.0-beta.2-manifest.json
SHA256SUMS
```

Only use assets attached to the `0.1.0-beta.2` release. Do not download a ZIP
of this documentation repository as a substitute for the application release.

Verify the archive checksum before extracting it.

macOS or Linux:

```bash
shasum -a 256 -c SHA256SUMS
```

Windows PowerShell:

```powershell
Get-FileHash .\flit-0.1.0-beta.2.zip -Algorithm SHA256
Get-Content .\SHA256SUMS
```

On Windows, compare the displayed ZIP hash with the corresponding value in
`SHA256SUMS`.

## 3. Extract and inspect the release

macOS or Linux:

```bash
mkdir -p flit-releases
unzip flit-0.1.0-beta.2.zip -d flit-releases
cd flit-releases/flit-0.1.0-beta.2
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force flit-releases | Out-Null
Expand-Archive .\flit-0.1.0-beta.2.zip -DestinationPath .\flit-releases
Set-Location .\flit-releases\flit-0.1.0-beta.2
```

Inspect the embedded manifest:

```bash
python3 -m json.tool release-manifest.json
```

Windows may use:

```powershell
python -m json.tool release-manifest.json
```

Confirm that `version` is `0.1.0-beta.2` and the `gateway` and `web` entries use
`acrflitpublic.azurecr.io/flit/` plus an immutable `@sha256:` digest.

## 4. Create local configuration and secrets

From the extracted release root:

```bash
cp src/deployment/docker/.env.example src/deployment/docker/.env
python3 scripts/setup_runtime_secrets.py
```

Windows PowerShell:

```powershell
Copy-Item src/deployment/docker/.env.example src/deployment/docker/.env
python scripts/setup_runtime_secrets.py
```

The interactive setup asks about:

1. bundled or external PostgreSQL;
2. database name, user, and volume;
3. generated or supplied secrets;
4. the Redis volume;
5. optional Microsoft Entra sign-in; and
6. optional Presidio services.

For a first local test, choose bundled PostgreSQL, accept `flit_pgdata` and
`flit_redisdata`, generate new secrets, and leave Microsoft Entra and Presidio
disabled unless they are part of your test assignment.

Generated credentials are stored as individual files under:

```text
src/deployment/docker/secrets/
```

Do not share these files or commit them to source control. Back them up only
through an approved secret-management process.

Validate the result without changing it:

```bash
python3 scripts/setup_runtime_secrets.py --validate-only
```

## 5. Review and pull the images

Enter the Compose directory:

```bash
cd src/deployment/docker
```

Validate and inspect the effective configuration:

```bash
docker compose config --quiet
docker compose config --images
```

The Web and Gateway entries must be digest-qualified references under
`acrflitpublic.azurecr.io/flit/`. They must not use `:latest`, `:dev`, or an
`aquila` repository.

Pull all selected images without signing into Azure:

```bash
docker compose pull
```

An authentication error for a Flit image means anonymous access or network
policy is not working as expected. Record the exact image reference and error,
but do not publish Docker credential files.

## 6. Apply database migrations

Run LiteLLM migrations first:

```bash
docker compose --profile migration run --rm litellm-migrate
```

Continue only after it exits successfully. Then run Flit migrations:

```bash
docker compose --profile migration run --rm flit-migrate
```

Do not start the application after a failed migration. Capture sanitized logs
and contact the Flit team before retrying destructive recovery steps.

## 7. Start Flit

```bash
docker compose up -d --wait --wait-timeout 300
docker compose ps
```

PostgreSQL, Redis, Gateway, Web, and Nginx should be running and healthy.
Presidio services appear only when enabled during setup.

If startup fails:

```bash
docker compose logs --tail=100 \
  db redis gateway web nginx presidio-analyzer presidio-anonymizer
```

It is normal for optional services that were not enabled to be absent.

## 8. Sign in

Open <http://localhost:4000>.

The default bootstrap username is `admin` unless it was changed during setup.
Read the generated password privately.

macOS:

```bash
pbcopy < secrets/flit_bootstrap_admin_password
```

Linux:

```bash
less secrets/flit_bootstrap_admin_password
```

Windows PowerShell:

```powershell
Get-Content secrets/flit_bootstrap_admin_password -Raw | Set-Clipboard
```

The first login requires a password change. If the browser reports an invalid
CSRF token after secrets were regenerated, clear site data for `localhost`,
reload, and sign in again. Avoid alternating between `localhost` and
`127.0.0.1` because browser cookies are host-specific.

Changing the bootstrap password file after migrations does not reset an
existing administrator account.

## 9. Perform a smoke test

In the administration UI:

1. Confirm the Overview and Models pages load.
2. Add a supported provider/model deployment using test credentials.
3. Create a virtual key authorized for the configured model or group.
4. Send a test request through Flit's Nginx endpoint.

Example request:

```bash
curl http://localhost:4000/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <virtual-key>' \
  -d '{
    "model": "<configured-model-or-group>",
    "messages": [{"role": "user", "content": "Reply with: Flit is ready"}]
  }'
```

Replace both placeholders. A healthy installation without a configured model
cannot complete inference requests.

## 10. Test persistence

Stop and restart the stack without deleting volumes:

```bash
docker compose down
docker compose up -d --wait --wait-timeout 300
docker compose ps
```

Confirm that login, configured deployments, and virtual keys still work.
`docker compose down` preserves named volumes. Do not add `--volumes` unless
permanent deletion is intentional.

## Routine commands

```bash
docker compose ps
docker compose logs --tail=100 gateway web nginx
docker compose restart gateway web nginx
docker compose down
docker compose up -d --wait --wait-timeout 300
```

Run commands from `src/deployment/docker` so Compose finds the generated `.env`
and selected overlay configuration.

## Completely remove a test installation

The following operation permanently removes the bundled PostgreSQL and Redis
data. Back up anything needed before proceeding.

First stop the application:

```bash
docker compose down
```

Review the exact volumes:

```bash
docker volume ls --filter name=flit
```

Only when permanent deletion is intended:

```bash
docker volume rm flit_pgdata flit_redisdata
```

Delete the extracted release directory and its secret files through your
operating system after the volumes are removed.

## Upgrade and rollback guidance

Do not extract a new release over an existing release directory.

1. Back up PostgreSQL and the current secret files.
2. Extract the new release beside the old version.
3. Review its manifest, release notes, configuration changes, and migrations.
4. Preserve existing secrets and copy only intentional non-secret settings.
5. Pull the new digest-qualified images.
6. Apply migrations in the documented order.
7. Stop the old Compose project and start the new one.

Application rollback can reuse the previous release ZIP and image digests, but
database migrations may prevent a safe rollback. Confirm the release-specific
rollback notes before downgrading.

## Reporting a beta problem

Provide:

- Flit version (`0.1.0-beta.2`);
- operating system and architecture;
- Docker and Compose versions;
- the exact failed step;
- expected and actual behavior;
- whether the installation is new or upgraded; and
- the smallest relevant sanitized log excerpt.

Never provide:

- `.env` contents;
- anything under `secrets/`;
- bootstrap passwords;
- virtual keys, provider API keys, or bearer tokens;
- session cookies or CSRF tokens; or
- database backups containing real data.

See [Support](../SUPPORT.md) and [Security](../SECURITY.md) as those beta
processes are finalized.
