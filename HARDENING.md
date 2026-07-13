<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--setup-uv/v8.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in .github/workflows/test.yml. This includes: (1) `${{ matrix.input.expected-version }}` in the 'Correct version gets installed' and 'Output has correct version' run blocks (test-specific-version job); (2) `${{ matrix.input.expected-version }}` in 'Correct version gets installed' run blocks (test-from-working-directory-version and test-version-file-version jobs); (3) `${{ matrix.expected-os }}` in the 'Verify cache key contains OS version' run block (test-cache-key-os-version job); (4) `${{ matrix.inputs.expected-cache-dir }}` in a run block (test-cache-local job); (5) `${{ matrix.inputs.expected-python-dir }}` in the 'Check Python dir is expected dir' run block (test-python-install-dir job); (6) `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` in the all-tests-passed run block. Any ${{ }} expression interpolated directly into a run: block is a script-injection risk regardless of the context it reads from.

Locations:

- `.github/workflows/test.yml:139`
- `.github/workflows/test.yml:145`
- `.github/workflows/test.yml:195`
- `.github/workflows/test.yml:225`
- `.github/workflows/test.yml:490`
- `.github/workflows/test.yml:720`
- `.github/workflows/test.yml:1000`
- `.github/workflows/test.yml:1085`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test.yml by moving ${{ }} expressions from run: shell blocks into env: blocks: (1) test-specific-version job: Moved matrix.input.expected-version to env var EXPECTED_VERSION in both 'Correct version gets installed' and 'Output has correct version' steps. (2) test-from-working-directory-version job: Moved matrix.input.expected-version to env var EXPECTED_VERSION. (3) test-version-file-version job: Moved matrix.input.expected-version to env var EXPECTED_VERSION. (4) test-cache-key-os-version job: Moved matrix.expected-os to env var EXPECTED_OS. (5) test-cache-local job: Moved matrix.inputs.expected-cache-dir to env var EXPECTED_CACHE_DIR. (6) test-python-install-dir job: Moved matrix.inputs.expected-python-dir to env var EXPECTED_PYTHON_DIR. (7) all-tests-passed job: Moved both boolean expressions to env vars ALL_PASSED and EXIT_CODE, replacing 'exit ${{ ... }}' with a conditional shell check.

