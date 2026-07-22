<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ ... }}` expressions into shell commands (rule a). This includes `${{ matrix.input.expected-version }}`, `${{ matrix.expected-os }}`, `${{ matrix.inputs.expected-cache-dir }}`, `${{ matrix.inputs.expected-python-dir }}`, and `${{ needs.*.result }}` expressions embedded directly in shell strings and `exit` commands. Even though matrix values are workflow-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, allowing shell metacharacters to be injected. Offending lines include:
- `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-specific-version, test-from-working-directory-version, test-version-file-version jobs)
- `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` (test-cache-key-os-version job)
- `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` (test-cache-local job)
- `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` (test-python-install-dir job)
- `echo "All jobs passed: ${{ !(contains(needs.*.result, ...)) }}"` and `exit ${{ (contains(needs.*.result, ...)) && 1 || 0 }}` (all-tests-passed job)

Locations:

- `.github/workflows/test.yml:115`
- `.github/workflows/test.yml:120`
- `.github/workflows/test.yml:175`
- `.github/workflows/test.yml:200`
- `.github/workflows/test.yml:432`
- `.github/workflows/test.yml:435`
- `.github/workflows/test.yml:643`
- `.github/workflows/test.yml:905`
- `.github/workflows/test.yml:975`
- `.github/workflows/test.yml:977`

### github-env-injection (severity: high)

In `.github/workflows/update-docs.yml`, the "Get tag info" step writes `$TAG_NAME` (derived from `$INPUT_TAG` which is `${{ inputs.tag }}`) to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. The bash regex validation `[[ ! "$TAG_NAME" =~ ^v[0-9]+\.[0-9]+\.[0-9]+$ ]]` uses `$` which in bash matches end-of-line rather than end-of-string, so a value containing an embedded newline (e.g. `v1.0.0\nmalicious=value`) can bypass the check and inject additional key=value pairs into `$GITHUB_OUTPUT`. The unsanitized write is: `echo "tag=$TAG_NAME" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/update-docs.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed github-env-injection in update-docs.yml by sanitizing TAG_NAME with printf/tr before writing to GITHUB_OUTPUT. Fixed script-injection in test.yml by moving all ${{ matrix.* }} and ${{ needs.* }} expressions from run: shell strings into env: blocks and referencing them as plain environment variables ($EXPECTED_VERSION, $EXPECTED_OS, $EXPECTED_CACHE_DIR, $EXPECTED_PYTHON_DIR, $ALL_PASSED). The all-tests-passed job's exit ${{ ... }} pattern was replaced with a proper if/exit 1 check using an env var.

