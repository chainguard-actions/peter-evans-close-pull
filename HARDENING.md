<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ ... }}` expressions are directly interpolated inside `run:` shell command strings in action.yml, violating sub-rule (a). This allows an attacker to inject arbitrary shell commands via user-controlled inputs.

Violating lines in the 'Set Parameters' step:
- Line 24: `if [ -n "${{ inputs.comment }}" ]; then` — inputs.comment interpolated directly into shell condition
- Line 25: `comment="--comment \"${{ inputs.comment }}\""` — inputs.comment interpolated into shell variable assignment
- Line 31: `if [ "${{ inputs.delete-branch }}" = true ]; then` — inputs.delete-branch interpolated directly

Violating lines in the 'Close Pull' step:
- Line 37: `gh pr close -R "${{ inputs.repository }}"` — inputs.repository interpolated directly
- Line 38: `${{ steps.params.outputs.comment }}` — steps output interpolated directly AND unquoted (sub-rule b)
- Line 39: `${{ steps.params.outputs.delete-branch }}` — steps output interpolated directly AND unquoted (sub-rule b)
- Line 40: `"${{ inputs.pull-request-number }}"` — inputs.pull-request-number interpolated directly

All these values should be passed via `env:` variables and referenced as quoted shell variables (e.g., `"$INPUT_COMMENT"`) instead.

Locations:

- `action.yml:24`
- `action.yml:25`
- `action.yml:31`
- `action.yml:37`
- `action.yml:38`
- `action.yml:39`
- `action.yml:40`

### github-env-injection (severity: high)

The 'Set Parameters' step writes values derived from untrusted inputs to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

- Line 28: `echo "$comment" >> $GITHUB_OUTPUT` — the shell variable `$comment` was constructed from `${{ inputs.comment }}` (line 25). A newline embedded in `inputs.comment` can inject additional key=value pairs into GITHUB_OUTPUT, allowing an attacker to override subsequent step outputs.
- Line 32: `echo delete-branch="--delete-branch" >> $GITHUB_OUTPUT` — this write is gated on `${{ inputs.delete-branch }}` being interpolated directly into the shell condition (line 31); the condition itself is injection-vulnerable.

The fix requires sanitizing the value before writing: `safe=$(printf '%s' "$comment" | tr -d '\n\r'); echo "comment=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `action.yml:27`
- `action.yml:28`
- `action.yml:29`
- `action.yml:32`

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

Fixed all findings in action.yml:

1. **Set Parameters step**: Moved `inputs.comment` to `INPUT_COMMENT` env var and `inputs.delete-branch` to `INPUT_DELETE_BRANCH` env var. Shell now references `$INPUT_COMMENT` and `$INPUT_DELETE_BRANCH`. Sanitized the comment with `printf '%s' "$INPUT_COMMENT" | tr -d '\n\r'` before writing to GITHUB_OUTPUT (github-env-injection fix).

2. **Close Pull step**: Moved `inputs.repository` → `INPUT_REPOSITORY`, `inputs.pull-request-number` → `INPUT_PULL_REQUEST_NUMBER`, `steps.params.outputs.comment` → `PARAM_COMMENT`, `steps.params.outputs.delete-branch` → `PARAM_DELETE_BRANCH` into the env: block. Since PARAM_COMMENT and PARAM_DELETE_BRANCH are argument lists (e.g., `--comment text` and `--delete-branch`), used the xargs-based null-delimited tokenization pattern with bash arrays and proper empty-value guards to safely expand them as separate arguments to `gh pr close`.

