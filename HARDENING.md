<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v7.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v7.5.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ ... }}` expressions inside shell command strings (sub-rule a). This includes `${{ matrix.input.expected-version }}`, `${{ matrix.expected-os }}`, `${{ matrix.inputs.expected-cache-dir }}`, `${{ matrix.inputs.expected-python-dir }}`, and `${{ !(contains(needs.*.result, ...)) }}` / `exit ${{ ... }}`. These values go through YAML template substitution before the shell sees them, allowing shell metacharacter injection if any matrix value contains special characters. Affected lines include:
- `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (test-specific-version, test-from-working-directory-version, test-version-file-version jobs)
- `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]` (test-specific-version job)
- `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` (test-cache-key-os-version job)
- `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` (test-cache-local job)
- `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` (test-python-install-dir job)
- `echo "All jobs passed: ${{ !(contains(needs.*.result, ...)) }}"` and `exit ${{ ... }}` (all-tests-passed job)
Fix: move these values into `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:108`
- `.github/workflows/test.yml:112`
- `.github/workflows/test.yml:157`
- `.github/workflows/test.yml:186`
- `.github/workflows/test.yml:390`
- `.github/workflows/test.yml:600`
- `.github/workflows/test.yml:850`
- `.github/workflows/test.yml:920`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test.yml by moving ${{ }} expressions from shell command strings into env: blocks:

1. test-specific-version job (2 steps): Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var for both 'Correct version gets installed' and 'Output has correct version' steps.

2. test-from-working-directory-version job: Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var.

3. test-version-file-version job: Moved `${{ matrix.input.expected-version }}` into `EXPECTED_VERSION` env var.

4. test-cache-key-os-version job: Moved `${{ matrix.expected-os }}` into `EXPECTED_OS` env var.

5. test-cache-local job: Moved `${{ matrix.inputs.expected-cache-dir }}` into `EXPECTED_CACHE_DIR` env var.

6. test-python-install-dir job: Moved `${{ matrix.inputs.expected-python-dir }}` into `EXPECTED_PYTHON_DIR` env var. Also fixed a bug in the original error message (was printing the variable name literally instead of its value).

7. all-tests-passed job: Replaced `echo "All jobs passed: ${{ ... }}"` and `exit ${{ ... }}` with an `ALL_PASSED` env var containing the boolean expression, then used a standard `if [ "$ALL_PASSED" != "true" ]; then exit 1; fi` shell construct.

