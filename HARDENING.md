<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v7.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v7.6.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ ... }}` expressions into shell commands, violating rule (a). This allows expression values to be parsed as shell syntax before the shell ever sees them.

1. `test-specific-version` job — "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` and "Output has correct version" step: `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]`
2. `test-from-working-directory-version` job — "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
3. `test-version-file-version` job — "Correct version gets installed" step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
4. `test-cache-key-os-version` job — "Verify cache key contains OS version" step: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"`
5. `test-cache-local` job — run step: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]`
6. `test-python-install-dir` job — "Check Python dir is expected dir" step: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]`
7. `all-tests-passed` job — "All tests passed" step: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}`

All of these embed `${{ ... }}` template expressions directly inside shell command strings. The values are substituted by the Actions runner before the shell parses the command, so any metacharacters in the values can alter the shell command.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:118`
- `.github/workflows/test.yml:163`
- `.github/workflows/test.yml:191`
- `.github/workflows/test.yml:222`
- `.github/workflows/test.yml:390`
- `.github/workflows/test.yml:395`
- `.github/workflows/test.yml:626`
- `.github/workflows/test.yml:631`
- `.github/workflows/test.yml:855`
- `.github/workflows/test.yml:893`
- `.github/workflows/test.yml:1053`
- `.github/workflows/test.yml:1054`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection issues in .github/workflows/test.yml by moving ${{ }} expressions from shell run: blocks into env: blocks and referencing them as plain environment variables:

1. test-specific-version job (lines 113, 118): Moved `${{ matrix.input.expected-version }}` to EXPECTED_VERSION env var in both 'Correct version gets installed' and 'Output has correct version' steps.

2. test-from-working-directory-version job (line 163): Moved `${{ matrix.input.expected-version }}` to EXPECTED_VERSION env var in 'Correct version gets installed' step.

3. test-version-file-version job (line 191): Moved `${{ matrix.input.expected-version }}` to EXPECTED_VERSION env var in 'Correct version gets installed' step.

4. test-cache-key-os-version job (lines 390, 395): Moved `${{ matrix.expected-os }}` to EXPECTED_OS env var in 'Verify cache key contains OS version' step.

5. test-cache-local job (lines 626, 631): Moved `${{ matrix.inputs.expected-cache-dir }}` to EXPECTED_CACHE_DIR env var.

6. test-python-install-dir job (line 855): Moved `${{ matrix.inputs.expected-python-dir }}` to EXPECTED_PYTHON_DIR env var in 'Check Python dir is expected dir' step. Also fixed a bug where the error message printed the variable name instead of its value.

7. all-tests-passed job (lines 1053, 1054): Replaced `exit ${{ ... }}` and `echo "... ${{ ... }}"` with env vars ALL_PASSED and ANY_FAILED, using a proper shell conditional `if [ "$ANY_FAILED" = "true" ]; then exit 1; fi`.

