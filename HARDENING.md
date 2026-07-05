<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv--/v8.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--setup-uv--/v8.3.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ matrix.* }}` and `${{ needs.*.result }}` expressions inside shell commands. This allows template substitution to inject arbitrary shell metacharacters before the shell parses the command. Affected steps:
- `test-specific-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-specific-version` job, 'Output has correct version' step: `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]`
- `test-from-working-directory-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-version-file-version` job, 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]`
- `test-cache-key-os-version` job, 'Verify cache key contains OS version' step: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "... ${{ matrix.expected-os }}"`
- `test-cache-local` job: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]`
- `test-python-install-dir` job: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]`
- `all-tests-passed` job: `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}`

Locations:

- `.github/workflows/test.yml:100`
- `.github/workflows/test.yml:105`
- `.github/workflows/test.yml:155`
- `.github/workflows/test.yml:178`
- `.github/workflows/test.yml:380`
- `.github/workflows/test.yml:560`
- `.github/workflows/test.yml:790`
- `.github/workflows/test.yml:860`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 8 script injection locations in .github/workflows/test.yml by moving ${{ matrix.* }} and ${{ needs.*.result }} expressions out of run: shell commands and into env: blocks. Changes made:
1. test-specific-version 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var
2. test-specific-version 'Output has correct version': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var
3. test-from-working-directory-version 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var
4. test-version-file-version 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var
5. test-cache-key-os-version 'Verify cache key contains OS version': moved ${{ matrix.expected-os }} to EXPECTED_OS env var
6. test-cache-local run step: moved ${{ matrix.inputs.expected-cache-dir }} to EXPECTED_CACHE_DIR env var
7. test-python-install-dir 'Check Python dir is expected dir': moved ${{ matrix.inputs.expected-python-dir }} to EXPECTED_PYTHON_DIR env var
8. all-tests-passed 'All tests passed': replaced exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }} with ALL_PASSED env var and shell if/exit logic

