<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **astral-sh--setup-uv/v8.0.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `.github/workflows/test.yml` directly interpolate `${{ ... }}` expressions inside shell command strings (rule (a) violation). Although `matrix.*` values are workflow-controlled and not directly attacker-supplied via PR, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting.

Affected steps and offending lines:

1. **test-specific-version** — `Correct version gets installed` step:
   ```
   if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]; then
   ```

2. **test-specific-version** — `Output has correct version` step:
   ```
   if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]; then
   ```

3. **test-from-working-directory-version** — `Correct version gets installed` step:
   ```
   if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]; then
   ```

4. **test-version-file-version** — `Correct version gets installed` step:
   ```
   if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]; then
   ```

5. **test-cache-key-os-version** — `Verify cache key contains OS version` step:
   ```
   if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]; then
   echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"
   ```

6. **test-cache-local** — unnamed `run:` step:
   ```
   if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]; then
   ```

7. **test-python-install-dir** — `Check Python dir is expected dir` step:
   ```
   if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]; then
   ```

8. **all-tests-passed** — `All tests passed` step:
   ```
   echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"
   exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}
   ```

Fix: move the values into `env:` variables and reference them as `"$ENV_VAR"` in the shell script.

Locations:

- `.github/workflows/test.yml:107`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:172`
- `.github/workflows/test.yml:196`
- `.github/workflows/test.yml:418`
- `.github/workflows/test.yml:422`
- `.github/workflows/test.yml:636`
- `.github/workflows/test.yml:869`
- `.github/workflows/test.yml:960`
- `.github/workflows/test.yml:963`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed 8 script injection instances in hardened/action/.github/workflows/test.yml by moving all ${{ }} expressions from run: shell scripts into env: blocks:

1. test-specific-version / 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var.
2. test-specific-version / 'Output has correct version': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var (added alongside existing UV_VERSION env var).
3. test-from-working-directory-version / 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var.
4. test-version-file-version / 'Correct version gets installed': moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var.
5. test-cache-key-os-version / 'Verify cache key contains OS version': moved both ${{ matrix.expected-os }} references to EXPECTED_OS env var.
6. test-cache-local / unnamed run step: moved ${{ matrix.inputs.expected-cache-dir }} to EXPECTED_CACHE_DIR env var.
7. test-python-install-dir / 'Check Python dir is expected dir': moved ${{ matrix.inputs.expected-python-dir }} to EXPECTED_PYTHON_DIR env var.
8. all-tests-passed / 'All tests passed': moved both needs.*.result expressions to ALL_PASSED and FAILED env vars; replaced 'exit ${{ ... }}' with shell if/else logic using the FAILED env var.

