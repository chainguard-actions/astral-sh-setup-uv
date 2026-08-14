<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v10.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v10.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ ... }}` expressions into shell command strings (rule a violation). This means the expression value is substituted into the shell script before the shell parses it, allowing shell metacharacter injection if the value is attacker-influenced.

Affected instances:
1. `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` — in `test-specific-version` job
2. `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` — in `test-from-working-directory-version` job
3. `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` — in `test-version-file-version` job
4. `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"` — in `test-cache-key-os-version` job
5. `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` — in `test-cache-local` job
6. `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` — in `test-python-install-dir` job
7. `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` — in `all-tests-passed` job

All `${{ ... }}` expressions should be moved to `env:` blocks and referenced as shell variables (e.g., `"$MATRIX_EXPECTED_VERSION"`) to prevent shell injection.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:114`
- `.github/workflows/test.yml:168`
- `.github/workflows/test.yml:193`
- `.github/workflows/test.yml:194`
- `.github/workflows/test.yml:460`
- `.github/workflows/test.yml:461`
- `.github/workflows/test.yml:672`
- `.github/workflows/test.yml:673`
- `.github/workflows/test.yml:936`
- `.github/workflows/test.yml:937`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 script injection instances in .github/workflows/test.yml by moving ${{ }} expressions from run: shell blocks into env: blocks and referencing them as plain environment variables:
1. test-specific-version 'Correct version gets installed': matrix.input.expected-version → EXPECTED_VERSION env var
2. test-specific-version 'Output has correct version': matrix.input.expected-version → EXPECTED_VERSION env var
3. test-from-working-directory-version 'Correct version gets installed': matrix.input.expected-version → EXPECTED_VERSION env var
4. test-version-file-version 'Correct version gets installed': matrix.input.expected-version → EXPECTED_VERSION env var
5. test-cache-key-os-version 'Verify cache key contains OS version': matrix.expected-os → EXPECTED_OS env var
6. test-cache-local run step: matrix.inputs.expected-cache-dir → EXPECTED_CACHE_DIR env var
7. test-python-install-dir 'Check Python dir is expected dir': matrix.inputs.expected-python-dir → EXPECTED_PYTHON_DIR env var
8. all-tests-passed 'All tests passed': complex needs.*.result expressions moved to ALL_PASSED env var with if/then exit logic replacing the direct exit ${{ ... }} pattern

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/update-docs.yml, added sanitization of TAG_NAME and COMMIT_SHA before writing to $GITHUB_OUTPUT. Both values are now passed through `printf '%s' "$VAR" | tr -d '\n\r'` to strip embedded newlines/carriage returns, storing the results in safe_tag and safe_sha variables which are then written to $GITHUB_OUTPUT instead of the raw values.

