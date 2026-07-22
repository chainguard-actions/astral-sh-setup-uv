<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.3.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ matrix.* }}` expressions inside shell command strings. These values flow through YAML template substitution before the shell processes them, allowing shell metacharacters to be injected. Affected lines include:
- Line ~139: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-specific-version job, 'Correct version gets installed' step)
- Line ~145: `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]` (test-specific-version job, 'Output has correct version' step)
- Line ~205: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-from-working-directory-version job)
- Line ~230: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-version-file-version job)
- Line ~539: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"*` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"` (cache key verification step)
- Line ~799: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` (test-cache-local-path job)
- Line ~1104: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` (python install dir test)
- Line ~1191: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}` (all-tests-passed job)
All `${{ ... }}` expressions must be moved to `env:` variables and the shell expansions must be double-quoted.

Locations:

- `.github/workflows/test.yml:139`
- `.github/workflows/test.yml:145`
- `.github/workflows/test.yml:205`
- `.github/workflows/test.yml:230`
- `.github/workflows/test.yml:539`
- `.github/workflows/test.yml:799`
- `.github/workflows/test.yml:1104`
- `.github/workflows/test.yml:1191`

### github-env-injection (severity: high)

In `.github/workflows/update-docs.yml`, the `Get tag info` step writes `TAG_NAME` to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. `TAG_NAME` is set from `$INPUT_TAG`, which is populated via `env: INPUT_TAG: ${{ inputs.tag }}` — a workflow-caller-controlled value. Although a regex check (`^v[0-9]+\.[0-9]+\.[0-9]+$`) is applied before the write, this does not substitute for newline sanitization. A value containing embedded newlines could inject additional key=value pairs into GITHUB_OUTPUT. The offending line is: `echo "tag=$TAG_NAME" >> "$GITHUB_OUTPUT"` (line ~40). The fix is to sanitize before writing: `safe=$(printf '%s' "$TAG_NAME" | tr -d '\n\r'); echo "tag=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/update-docs.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 8 script-injection locations in .github/workflows/test.yml by moving ${{ matrix.* }} and ${{ needs.*.result }} expressions from run: shell blocks into env: variables (EXPECTED_VERSION, EXPECTED_OS, EXPECTED_CACHE_DIR, EXPECTED_PYTHON_DIR, ALL_PASSED, EXIT_CODE). Fixed 1 github-env-injection in .github/workflows/update-docs.yml by adding printf '%s' ... | tr -d '\n\r' sanitization for both TAG_NAME and COMMIT_SHA before writing to $GITHUB_OUTPUT.

