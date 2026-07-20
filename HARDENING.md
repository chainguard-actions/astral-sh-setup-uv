<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v3.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v3.2.4** was hardened automatically. 18 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A ${{ secrets.GITHUB_TOKEN }} expression is interpolated directly inside a run: shell command string on line 17 of update-known-checksums.yml. Any ${{ ... }} expression in a run: block is a script-injection risk because YAML template substitution happens before the shell sees the value. The offending line is: `node dist/update-known-checksums/index.js src/download/checksum/known-checksums.ts ${{ secrets.GITHUB_TOKEN }}`. The token should be passed via an env: variable instead.

Locations:

- `.github/workflows/update-known-checksums.yml:17`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or version strings instead of immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks. Unpinned references: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4.

Locations:

- `.github/workflows/check-dist.yml:18`
- `.github/workflows/check-dist.yml:22`
- `.github/workflows/check-dist.yml:43`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or version strings instead of immutable 40-character commit SHAs. Unpinned references: actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3.

Locations:

- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:55`

### unpinned-uses (severity: high)

Workflow references release-drafter/release-drafter@v6.0.0 by a mutable version tag instead of an immutable 40-character commit SHA.

Locations:

- `.github/workflows/release-drafter.yml:15`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 by a mutable tag instead of an immutable 40-character commit SHA.

Locations:

- `.github/workflows/test-cache-windows.yml:18`
- `.github/workflows/test-cache-windows.yml:37`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 by a mutable tag instead of an immutable 40-character commit SHA.

Locations:

- `.github/workflows/test-cache.yml:19`
- `.github/workflows/test-cache.yml:38`
- `.github/workflows/test-cache.yml:62`
- `.github/workflows/test-cache.yml:79`
- `.github/workflows/test-cache.yml:103`
- `.github/workflows/test-cache.yml:119`
- `.github/workflows/test-cache.yml:141`
- `.github/workflows/test-cache.yml:155`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 by a mutable tag instead of an immutable 40-character commit SHA.

Locations:

- `.github/workflows/test-windows.yml:18`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 and actions/setup-node@v4 by mutable tags instead of immutable 40-character commit SHAs.

Locations:

- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:17`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 and actions/setup-node@v4 by mutable tags instead of immutable 40-character commit SHAs.

Locations:

- `.github/workflows/update-known-checksums.yml:9`
- `.github/workflows/update-known-checksums.yml:10`

### unpinned-uses (severity: high)

Workflow references actions/checkout@v4 and haya14busa/action-update-semver@v1.2.1 by mutable tags instead of immutable 40-character commit SHAs.

Locations:

- `.github/workflows/update-major-minor-tags.yml:16`
- `.github/workflows/update-major-minor-tags.yml:18`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/check-dist.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/release-drafter.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/test-cache-windows.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/test-cache.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/test-windows.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/test.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/update-known-checksums.yml:1`

### missing-permissions (severity: medium)

Workflow file has no top-level permissions: key and no job-level permissions: keys on any job. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. Add a top-level permissions: block with minimal required scopes.

Locations:

- `.github/workflows/update-major-minor-tags.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 17 findings across 8 workflow files:

1. script-injection (update-known-checksums.yml line 17): Moved `${{ secrets.GITHUB_TOKEN }}` from the run: shell command into an env: block as GITHUB_TOKEN, then referenced it as "$GITHUB_TOKEN" in the shell script.

2. unpinned-uses: Pinned all mutable action tags to immutable commit SHAs:
   - actions/checkout@v4 → 11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → 49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/upload-artifact@v4 → ea165f8d65b6e75b540449e92b4886f43607fa02
   - github/codeql-action/init@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - github/codeql-action/autobuild@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - github/codeql-action/analyze@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - release-drafter/release-drafter@v6.0.0 → 3f0f87098bd6b5c5b9a36d49c41d998ea58f9348
   - haya14busa/action-update-semver@v1.2.1 → fb48464b2438ae82cc78237be61afb4f461265a1

3. missing-permissions: Added top-level permissions blocks to all 7 affected workflows with minimal required scopes (contents: read for test workflows; contents: write and/or pull-requests: write for workflows that create PRs or update tags).

