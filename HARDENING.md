<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--setup-uv/v8.3.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ matrix.* }}` and `${{ needs.*.result }}` expressions into shell commands (rule a). These values flow through YAML template substitution before the shell processes them, enabling script injection. Affected steps:
- `test-specific-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-specific-version` job, 'Output has correct version' step: `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]`
- `test-from-working-directory-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-version-file-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-cache-key-os-version` job, 'Verify cache key contains OS version' step: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"`
- `test-cache-local` job, run step: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]`
- `test-python-install-dir` job, 'Check Python dir is expected dir' step: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]`
- `all-tests-passed` job: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}`

Locations:

- `.github/workflows/test.yml:121`
- `.github/workflows/test.yml:127`
- `.github/workflows/test.yml:167`
- `.github/workflows/test.yml:185`
- `.github/workflows/test.yml:449`
- `.github/workflows/test.yml:451`
- `.github/workflows/test.yml:672`
- `.github/workflows/test.yml:920`
- `.github/workflows/test.yml:1054`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 9 script injection locations in .github/workflows/test.yml by moving ${{ matrix.* }} and ${{ needs.*.result }} expressions from run: shell blocks into env: blocks:
1. test-specific-version/'Correct version gets installed': EXPECTED_VERSION env var
2. test-specific-version/'Output has correct version': EXPECTED_VERSION env var added to existing env block
3. test-from-working-directory-version/'Correct version gets installed': EXPECTED_VERSION env var
4. test-version-file-version/'Correct version gets installed': EXPECTED_VERSION env var
5. test-cache-key-os-version/'Verify cache key contains OS version': EXPECTED_OS env var (both echo and if condition)
6. test-cache-local/run step: EXPECTED_CACHE_DIR env var
7. test-python-install-dir/'Check Python dir is expected dir': EXPECTED_PYTHON_DIR env var
8. all-tests-passed/'All tests passed': ALL_PASSED env var, replaced exit ${{ ... }} with if/exit 1 pattern

