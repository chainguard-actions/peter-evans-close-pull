<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.1** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml. In the 'Set Parameters' step: `if [ -n "${{ inputs.comment }}" ]`, `comment="--comment \"${{ inputs.comment }}\""`, and `if [ "${{ inputs.delete-branch }}" = true ]` all embed untrusted inputs directly into shell code before the shell ever sees them, enabling command injection. In the 'Close Pull' step: `gh pr close -R "${{ inputs.repository }}"`, the unquoted `${{ steps.params.outputs.comment }}`, the unquoted `${{ steps.params.outputs.delete-branch }}`, and `"${{ inputs.pull-request-number }}"` all interpolate expressions directly into the shell command string.

Locations:

- `action.yml:23`
- `action.yml:24`
- `action.yml:30`
- `action.yml:37`
- `action.yml:38`
- `action.yml:39`
- `action.yml:40`

### github-env-injection (severity: high)

In the 'Set Parameters' step, the untrusted value `${{ inputs.comment }}` is interpolated directly into the shell variable `comment` (line 24), which is then written to $GITHUB_OUTPUT via `echo "$comment" >> $GITHUB_OUTPUT` (line 27) without any sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled comment containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:27`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.comment }}" appears directly in run: block of step "Set Parameters"; move to env: map

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.comment }}" appears directly in run: block of step "Set Parameters"; move to env: map

Locations:

- `action.yml:26`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.delete-branch }}" appears directly in run: block of step "Set Parameters"; move to env: map

Locations:

- `action.yml:32`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repository }}" appears directly in run: block of step "Close Pull"; move to env: map

Locations:

- `action.yml:39`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pull-request-number }}" appears directly in run: block of step "Close Pull"; move to env: map

Locations:

- `action.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to fix all injection findings:

1. **script-injection / static-inline-injection**: Moved all ${{ }} expressions from run: blocks into env: maps. 'Set Parameters' step now has `INPUT_COMMENT` and `INPUT_DELETE_BRANCH` env vars; 'Close Pull' step now has `INPUT_REPOSITORY`, `INPUT_PULL_REQUEST_NUMBER`, `PARAM_COMMENT`, `PARAM_DELETE_BRANCH`, and `GH_TOKEN` env vars. Shell scripts reference plain `$VAR` names.

2. **github-env-injection**: The comment input is sanitized with `printf '%s' "$INPUT_COMMENT" | tr -d '\n\r'` before writing to GITHUB_OUTPUT, preventing newline-based injection of additional key=value pairs.

3. **Structural improvement**: The 'Close Pull' step now uses a bash array (`args=()`) to build optional flags (`--comment` and `--delete-branch`), avoiding the single-word trap. The comment output stores just the text (not the flag+value), and the flag is added separately in the array.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed action.yml line 29-32: Removed the `safe_comment` variable that was stripping newlines from `$INPUT_COMMENT` before writing it as the body of a heredoc-delimited GITHUB_OUTPUT block. Since the random delimiter already prevents injection, newlines should not be stripped from heredoc-delimited values (doing so merges multi-line content into one line). Now writes `$INPUT_COMMENT` directly in the heredoc body.

