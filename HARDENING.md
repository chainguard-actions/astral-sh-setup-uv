<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ ... }}` expressions into shell command strings (sub-rule a). Although `matrix.*` and `needs.*.result` are not attacker-controlled in the same way as `github.event.*`, any `${{ }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell string before the shell parses it, bypassing quoting. Affected lines include:
- `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (jobs: test-specific-version, test-from-working-directory-version, test-version-file-version)
- `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]` (job: test-specific-version)
- `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` (job: test-cache-key-os-version)
- `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` (job: test-cache-local)
- `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` (job: test-python-install-dir)
- `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}` (job: all-tests-passed)
Fix: move all `${{ }}` values into `env:` variables and reference them as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:120`
- `.github/workflows/test.yml:175`
- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:456`
- `.github/workflows/test.yml:672`
- `.github/workflows/test.yml:906`
- `.github/workflows/test.yml:1048`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 8 script injection locations in .github/workflows/test.yml by moving ${{ }} expressions from run: shell blocks into env: blocks and referencing them as plain environment variables ($VAR_NAME) in the shell scripts. Specific fixes:
1. test-specific-version 'Correct version gets installed': ${{ matrix.input.expected-version }} → env: EXPECTED_VERSION
2. test-specific-version 'Output has correct version': ${{ matrix.input.expected-version }} → env: EXPECTED_VERSION
3. test-from-working-directory-version 'Correct version gets installed': ${{ matrix.input.expected-version }} → env: EXPECTED_VERSION
4. test-version-file-version 'Correct version gets installed': ${{ matrix.input.expected-version }} → env: EXPECTED_VERSION
5. test-cache-key-os-version 'Verify cache key contains OS version': ${{ matrix.expected-os }} → env: EXPECTED_OS
6. test-cache-local run step: ${{ matrix.inputs.expected-cache-dir }} → env: EXPECTED_CACHE_DIR
7. test-python-install-dir 'Check Python dir is expected dir': ${{ matrix.inputs.expected-python-dir }} → env: EXPECTED_PYTHON_DIR
8. all-tests-passed 'All tests passed': both needs.*.result expressions → env: ALL_PASSED and EXIT_CODE

