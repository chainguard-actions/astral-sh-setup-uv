<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v10.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v10.0.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/test.yml directly interpolate ${{ matrix.* }} expressions inside shell commands (rule a). These expressions flow through YAML template substitution before the shell processes them, enabling script injection. Affected steps include:
- 'Correct version gets installed': `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- 'Output has correct version': `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]`
- Cache key OS check: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"`
- Cache dir check: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]`
- Python dir check: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]`
- all-tests-passed job: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}`
These should be moved to env: variables and the shell expansions double-quoted.

Locations:

- `.github/workflows/test.yml:110`
- `.github/workflows/test.yml:116`
- `.github/workflows/test.yml:170`
- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:468`
- `.github/workflows/test.yml:471`
- `.github/workflows/test.yml:676`
- `.github/workflows/test.yml:930`
- `.github/workflows/test.yml:1020`
- `.github/workflows/test.yml:1023`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test.yml by moving ${{ matrix.* }} and ${{ needs.*.result }} expressions from run: shell script bodies into env: blocks. Changes made:
1. test-specific-version 'Correct version gets installed': moved matrix.input.expected-version to EXPECTED_VERSION env var
2. test-specific-version 'Output has correct version': added EXPECTED_VERSION env var alongside existing UV_VERSION
3. test-from-working-directory-version 'Correct version gets installed': moved matrix.input.expected-version to EXPECTED_VERSION env var
4. test-version-file-version 'Correct version gets installed': moved matrix.input.expected-version to EXPECTED_VERSION env var
5. test-cache-key-os-version 'Verify cache key contains OS version': moved matrix.expected-os to EXPECTED_OS env var
6. test-cache-local: moved matrix.inputs.expected-cache-dir to EXPECTED_CACHE_DIR env var
7. test-python-install-dir 'Check Python dir is expected dir': moved matrix.inputs.expected-python-dir to EXPECTED_PYTHON_DIR env var (also fixed a bug where the error message printed the variable name instead of its value)
8. all-tests-passed 'All tests passed': moved both needs.*.result expressions to ALL_PASSED and EXIT_CODE env vars

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/update-docs.yml. In the 'Get tag info' step, added sanitization using `printf '%s' "$TAG_NAME" | tr -d '\n\r'` and `printf '%s' "$COMMIT_SHA" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT. The sanitized values are stored in `safe_tag` and `safe_sha` variables respectively, which are then written to $GITHUB_OUTPUT instead of the raw values.

