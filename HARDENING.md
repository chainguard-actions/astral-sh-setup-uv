<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v3.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--setup-uv/v3.2.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned SHA digests, making them vulnerable to supply-chain attacks. Unpinned references found:
- check-dist.yml: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4
- codeql-analysis.yml: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3
- release-drafter.yml: release-drafter/release-drafter@v6.0.0
- test-cache-windows.yml: actions/checkout@v4
- test-cache.yml: actions/checkout@v4
- test-windows.yml: actions/checkout@v4
- test.yml: actions/checkout@v4, actions/setup-node@v4
- update-known-checksums.yml: actions/checkout@v4, actions/setup-node@v4
- update-major-minor-tags.yml: actions/checkout@v4, haya14busa/action-update-semver@v1.2.1

Locations:

- `.github/workflows/check-dist.yml:19`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:41`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/release-drafter.yml:15`
- `.github/workflows/test-cache-windows.yml:14`
- `.github/workflows/test-cache.yml:15`
- `.github/workflows/test-windows.yml:14`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-known-checksums.yml:9`
- `.github/workflows/update-known-checksums.yml:10`
- `.github/workflows/update-major-minor-tags.yml:13`
- `.github/workflows/update-major-minor-tags.yml:15`

### missing-permissions (severity: medium)

The following workflow files have no top-level 'permissions:' key and no job-level 'permissions:' on any job, meaning they run with the default (potentially broad) token permissions: check-dist.yml, release-drafter.yml, test-cache-windows.yml, test-cache.yml, test-windows.yml, test.yml, update-known-checksums.yml, update-major-minor-tags.yml. Only codeql-analysis.yml has explicit permissions defined.

Locations:

- `.github/workflows/check-dist.yml:1`
- `.github/workflows/release-drafter.yml:1`
- `.github/workflows/test-cache-windows.yml:1`
- `.github/workflows/test-cache.yml:1`
- `.github/workflows/test-windows.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-known-checksums.yml:1`
- `.github/workflows/update-major-minor-tags.yml:1`

### script-injection (severity: high)

Sub-rule (a): A ${{ ... }} expression is interpolated directly inside a run: shell command string. In update-known-checksums.yml, the run: block passes '${{ secrets.GITHUB_TOKEN }}' as a CLI argument directly in the shell command: 'node dist/update-known-checksums/index.js src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}'. Any ${{ }} expression inside a run: block is a script-injection risk as the value is substituted by the template engine before the shell processes it.

Locations:

- `.github/workflows/update-known-checksums.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 8 workflow files:

1. unpinned-uses: Pinned all action references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02
   - github/codeql-action/init@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - github/codeql-action/autobuild@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - github/codeql-action/analyze@v3 → @02c5e83432fe5497fd85b873b6c9f16a8578e1d9
   - release-drafter/release-drafter@v6.0.0 → @3f0f87098bd6b5c5b9a36d49c41d998ea58f9348
   - haya14busa/action-update-semver@v1.2.1 → @fb48464b2438ae82cc78237be61afb4f461265a1

2. missing-permissions: Added top-level permissions blocks to all 8 affected workflow files:
   - check-dist.yml: contents: read
   - release-drafter.yml: contents: write, pull-requests: read
   - test-cache-windows.yml: contents: read
   - test-cache.yml: contents: read
   - test-windows.yml: contents: read
   - test.yml: contents: read
   - update-known-checksums.yml: contents: write, pull-requests: write
   - update-major-minor-tags.yml: contents: write

3. script-injection: In update-known-checksums.yml, moved ${{ secrets.GITHUB_TOKEN }} out of the run: shell command and into the step's env: block, then referenced it as $GITHUB_TOKEN in the shell script.

