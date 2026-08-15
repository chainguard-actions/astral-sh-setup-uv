<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in violation of sub-rule (a). This includes `${{ matrix.input.expected-version }}`, `${{ matrix.expected-os }}`, `${{ matrix.inputs.expected-cache-dir }}`, `${{ matrix.inputs.expected-python-dir }}`, `${{ needs.*.result }}`, and related expressions. Any `${{ ... }}` inside a `run:` block is a script-injection risk because YAML template substitution happens before the shell ever sees the string, allowing an attacker who controls matrix values or needs results to inject shell metacharacters. Offending lines include:
- `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]; then` (test-specific-version, test-from-working-directory-version, test-version-file-version jobs)
- `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]; then` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"` (test-cache-key-os-version job)
- `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]; then` (test-cache-local job)
- `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]; then` (test-python-install-dir job)
- `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || ...) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }}` (all-tests-passed job)

Locations:

- `.github/workflows/test.yml:139`
- `.github/workflows/test.yml:145`
- `.github/workflows/test.yml:206`
- `.github/workflows/test.yml:240`
- `.github/workflows/test.yml:340`
- `.github/workflows/test.yml:341`
- `.github/workflows/test.yml:490`
- `.github/workflows/test.yml:580`
- `.github/workflows/test.yml:731`
- `.github/workflows/test.yml:733`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test.yml by moving ${{ }} expressions from run: shell script strings into env: blocks:

1. test-specific-version job: Moved ${{ matrix.input.expected-version }} to env: EXPECTED_VERSION in both 'Correct version gets installed' and 'Output has correct version' steps
2. test-from-working-directory-version job: Moved ${{ matrix.input.expected-version }} to env: EXPECTED_VERSION
3. test-version-file-version job: Moved ${{ matrix.input.expected-version }} to env: EXPECTED_VERSION
4. test-cache-key-os-version job: Moved ${{ matrix.expected-os }} to env: EXPECTED_OS and ${{ steps.setup-uv.outputs.cache-key }} to env: CACHE_KEY (consolidated env block)
5. test-cache-local job: Moved ${{ matrix.inputs.expected-cache-dir }} to env: EXPECTED_CACHE_DIR
6. test-python-install-dir job: Moved ${{ matrix.inputs.expected-python-dir }} to env: EXPECTED_PYTHON_DIR
7. all-tests-passed job: Replaced the complex ${{ (contains(needs.*.result, 'failure') || ...) && 1 || 0 }} expression with ${{ join(needs.*.result, ',') }} in env: NEEDS_RESULTS and used bash grep logic to check for failures/cancellations

