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
