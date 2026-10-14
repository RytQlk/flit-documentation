# Flit releases

> **Work in progress:** This guide will define release channels, compatibility,
> release notes, checksums, upgrade requirements, rollback constraints, and
> support windows.

During beta, always install from a versioned GitHub Release asset and retain its
manifest with the release ZIP.

Each release provides a versioned manifest beside the ZIP and embeds the same
data as `release-manifest.json` inside the archive. The external file lets
operators and automation inspect version, source revisions, supported
platforms, and exact image digests without downloading the ZIP. The embedded
copy keeps an extracted installation self-describing if it is later separated
from the GitHub release page.

The embedded manifest is sufficient for installation. The external copy is a
metadata convenience, not by itself proof of ZIP integrity. Verify the ZIP
against the release's `SHA256SUMS` before extraction.

## 0.1.0-beta.4

This release simplifies the public Docker configuration contract and adds a
safe post-installation reconfiguration workflow.

- New installations use `flit` as the default PostgreSQL database and user.
- Setup recommends Gateway workers from Docker's available CPU count, bounded
  to 1–8; Web workers retain the default of two.
- Managed Presidio remains optional during setup and can be enabled later.
- Initial setup refuses to overwrite an existing `.env`.
- `--reconfigure` supports bounded updates to Gateway workers, Presidio,
  Microsoft Entra SSO, and Azure AI Search retrieval.
- Reconfiguration validates a complete candidate before applying it, supports
  deferred `--apply-reconfigure`, backs up the active configuration, restores
  it after a failed restart, rejects stale candidates, and records redacted
  local troubleshooting history.
- Branding and legacy compatibility controls remain product capabilities but
  are excluded from the standard public release configuration.
- Pgweb is excluded from the standard release topology and remains available
  only as a loopback-bound source-development diagnostic.

Existing installations retain their configured database identity, volumes,
and secrets. Review the installation guide before upgrading and back up
PostgreSQL and the secret directory first.
