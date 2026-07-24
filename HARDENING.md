<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ ... }}` expressions inside shell commands (sub-rule a). This allows the expression value to be parsed by the shell before quoting can protect it.

1. `test-specific-version` job, "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
2. `test-specific-version` job, "Output has correct version" step: `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]`
3. `test-from-working-directory-version` job, "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
4. `test-version-file-version` job, "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
5. `test-cache-key-os-version` job, "Verify cache key contains OS version" step: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"`
6. `test-cache-local` job: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]`
7. `test-python-install-dir` job, "Check Python dir is expected dir" step: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]`
8. `all-tests-passed` job, "All tests passed" step: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}`

All `${{ ... }}` expressions inside `run:` blocks are script-injection risks regardless of whether the context appears GitHub-controlled.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:118`
- `.github/workflows/test.yml:175`
- `.github/workflows/test.yml:200`
- `.github/workflows/test.yml:440`
- `.github/workflows/test.yml:641`
- `.github/workflows/test.yml:900`
- `.github/workflows/test.yml:975`

### github-env-injection (severity: high)

In `update-docs.yml`, the "Get tag info" step writes `TAG_NAME` to `$GITHUB_OUTPUT` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. `TAG_NAME` is derived from `$INPUT_TAG`, which is set from `${{ inputs.tag }}` (a workflow-controllable input). Although a regex validation (`^v[0-9]+\.[0-9]+\.[0-9]+$`) is applied before the write, the check requires explicit newline stripping via `tr -d '\n\r'` before every write to a special environment file when the source is untrusted input. The lines `echo "tag=$TAG_NAME" >> "$GITHUB_OUTPUT"` and `echo "sha=$COMMIT_SHA" >> "$GITHUB_OUTPUT"` are missing this sanitization step.

Locations:

- `.github/workflows/update-docs.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed github-env-injection in update-docs.yml by sanitizing TAG_NAME and COMMIT_SHA with printf/tr before writing to $GITHUB_OUTPUT. Fixed all 8 script-injection instances in test.yml by moving ${{ ... }} expressions from run: shell commands into env: blocks and referencing them as plain environment variables ($EXPECTED_VERSION, $EXPECTED_OS, $EXPECTED_CACHE_DIR, $EXPECTED_PYTHON_DIR, $ALL_PASSED, $EXIT_CODE).

