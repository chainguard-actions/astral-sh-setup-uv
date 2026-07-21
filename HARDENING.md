<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v9.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v9.0.0** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Multiple `${{ matrix.* }}` expressions are directly interpolated inside `run:` shell commands in test.yml. This allows shell metacharacters in matrix values to be interpreted by the shell before quoting can occur.

1. `${{ matrix.input.expected-version }}` directly in shell: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-specific-version, test-from-working-directory-version, test-version-file-version jobs)
2. `${{ matrix.expected-os }}` directly in shell: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "...version: ${{ matrix.expected-os }}"` (test-cache-key-os-version job)
3. `${{ matrix.inputs.expected-python-dir }}` directly in shell: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` (test-python-install-dir job)
4. `${{ !(contains(needs.*.result, 'failure') || ...) }}` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}` directly in shell (all-tests-passed job)

Fix: Move matrix values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:119`
- `.github/workflows/test.yml:124`
- `.github/workflows/test.yml:183`
- `.github/workflows/test.yml:213`
- `.github/workflows/test.yml:218`
- `.github/workflows/test.yml:476`
- `.github/workflows/test.yml:481`
- `.github/workflows/test.yml:979`
- `.github/workflows/test.yml:984`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test.yml by moving ${{ }} expressions out of run: shell commands and into env: blocks:

1. test-specific-version job (lines ~119, 124): Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var in both 'Correct version gets installed' and 'Output has correct version' steps.

2. test-from-working-directory-version job (line ~183): Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var.

3. test-version-file-version job: Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var.

4. test-cache-key-os-version job (lines ~213, 218): Moved `${{ matrix.expected-os }}` into `EXPECTED_OS` env var in the 'Verify cache key contains OS version' step.

5. test-cache-local job: Moved `${{ matrix.inputs.expected-cache-dir }}` into `EXPECTED_CACHE_DIR` env var (additional finding not in original list).

6. test-python-install-dir job (lines ~476, 481): Moved `${{ matrix.inputs.expected-python-dir }}` into `EXPECTED_PYTHON_DIR` env var.

7. all-tests-passed job (lines ~979, 984): Moved `${{ !(contains(needs.*.result, 'failure') || ...) }}` into `ALL_PASSED` env var and `${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}` into `EXIT_CODE` env var.

All shell scripts now reference these values as plain `$VAR_NAME` environment variables, preventing shell metacharacter injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in .github/workflows/update-docs.yml. In the 'Get tag info' step, both TAG_NAME and COMMIT_SHA are now sanitized using `printf '%s' "$VAR" | tr -d '\n\r'` before being written to $GITHUB_OUTPUT. The sanitized values are stored in safe_tag and safe_sha variables respectively, which are then used in the echo statements. This prevents newline injection attacks where an attacker-controlled `tag` input could inject newlines to poison GITHUB_OUTPUT.

