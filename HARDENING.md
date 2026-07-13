<!-- markdownlint-disable -->

# Hardening Report: astral-sh--setup-uv/v8.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **astral-sh--setup-uv/v8.3.2** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): In the `test-specific-version` job, two run: steps directly interpolate `${{ matrix.input.expected-version }}` inside shell commands: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` (Correct version gets installed step) and `if [ "$UV_VERSION" != "${{ matrix.input.expected-version }}" ]` (Output has correct version step). The `matrix.*` context is an untrusted-input source — any `${{ ... }}` directly inside a run: script is a script-injection finding.

Locations:

- `.github/workflows/test.yml:116`
- `.github/workflows/test.yml:121`

### script-injection (severity: high)

Sub-rule (a): In the `test-from-working-directory-version` job's 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` — direct interpolation of `${{ matrix.input.expected-version }}` in a run: block.

Locations:

- `.github/workflows/test.yml:163`

### script-injection (severity: high)

Sub-rule (a): In the `test-version-file-version` job's 'Correct version gets installed' step: `if [ "$(uv --version)" != "uv ${{ matrix.input.expected-version }}" ]` — direct interpolation of `${{ matrix.input.expected-version }}` in a run: block.

Locations:

- `.github/workflows/test.yml:185`

### script-injection (severity: high)

Sub-rule (a): In the `test-cache-key-os-version` job's 'Verify cache key contains OS version' step: `if [[ "$CACHE_KEY" != *"${{ matrix.expected-os }}"* ]]` and `echo "Cache key does not contain expected OS version: ${{ matrix.expected-os }}"` — direct interpolation of `${{ matrix.expected-os }}` in a run: block.

Locations:

- `.github/workflows/test.yml:399`

### script-injection (severity: high)

Sub-rule (a): In the `test-cache-local` job's run step: `if [ "$UV_CACHE_DIR" != "${{ matrix.inputs.expected-cache-dir }}" ]` — direct interpolation of `${{ matrix.inputs.expected-cache-dir }}` in a run: block.

Locations:

- `.github/workflows/test.yml:586`

### script-injection (severity: high)

Sub-rule (a): In the `test-python-install-dir` job's 'Check Python dir is expected dir' step: `if [ "$UV_PYTHON_INSTALL_DIR" != "${{ matrix.inputs.expected-python-dir }}" ]` — direct interpolation of `${{ matrix.inputs.expected-python-dir }}` in a run: block.

Locations:

- `.github/workflows/test.yml:808`

### script-injection (severity: high)

Sub-rule (a): In the `all-tests-passed` job's 'All tests passed' step: `echo "All jobs passed: ${{ !(contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) }}"` and `exit ${{ (contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')) && 1 || 0 }}` — direct interpolation of `${{ needs.*.result }}` expressions in a run: block. The `needs.*` context is an untrusted-input source.

Locations:

- `.github/workflows/test.yml:882`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed 7 script-injection findings in hardened/action/.github/workflows/test.yml by moving all ${{ }} expressions out of run: blocks and into env: blocks:
1. test-specific-version job (lines 116, 121): Moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var in both 'Correct version gets installed' and 'Output has correct version' steps.
2. test-from-working-directory-version job (line 163): Moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var.
3. test-version-file-version job (line 185): Moved ${{ matrix.input.expected-version }} to EXPECTED_VERSION env var.
4. test-cache-key-os-version job (line 399): Moved ${{ matrix.expected-os }} to EXPECTED_OS env var in 'Verify cache key contains OS version' step.
5. test-cache-local job (line 586): Moved ${{ matrix.inputs.expected-cache-dir }} to EXPECTED_CACHE_DIR env var.
6. test-python-install-dir job (line 808): Moved ${{ matrix.inputs.expected-python-dir }} to EXPECTED_PYTHON_DIR env var. Also fixed a bug in the original error message (was printing literal 'UV_PYTHON_INSTALL_DIR' instead of the variable value).
7. all-tests-passed job (line 882): Moved ${{ needs.*.result }} expressions to ALL_PASSED and EXIT_CODE env vars, then referenced them as $ALL_PASSED and $EXIT_CODE in the shell script.

