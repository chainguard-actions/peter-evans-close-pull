<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In action.yml 'Set Parameters' step, ${{ inputs.comment }} and ${{ inputs.delete-branch }} are interpolated directly into shell commands. In the 'Close Pull' step, ${{ inputs.repository }}, ${{ steps.params.outputs.comment }}, ${{ steps.params.outputs.delete-branch }}, and ${{ inputs.pull-request-number }} are all interpolated directly into the gh pr close command. In update-major-version.yml, ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} are interpolated directly into git tag and git push run: commands, allowing an attacker to inject arbitrary shell commands via workflow_dispatch inputs.

Locations:

- `action.yml:22`
- `action.yml:35`
- `.github/workflows/update-major-version.yml:18`
- `.github/workflows/update-major-version.yml:20`

### github-env-injection (severity: high)

In action.yml 'Set Parameters' step, ${{ inputs.comment }} is interpolated into the shell variable 'comment' and then written to $GITHUB_OUTPUT via 'echo "$comment" >> $GITHUB_OUTPUT' without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled newline in inputs.comment could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps. Similarly, ${{ inputs.delete-branch }} controls a write to $GITHUB_OUTPUT without sanitization.

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of immutable full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: automerge-dependabot.yml: 'peter-evans/enable-pull-request-automerge@v3'; ci.yml: 'actions/checkout@v3', 'actions/upload-artifact@v3', 'peter-evans/create-pull-request@v5', 'actions/download-artifact@v3'; slash-command-dispatch.yml: 'peter-evans/slash-command-dispatch@v3'; update-major-version.yml: 'actions/checkout@v3'.

Locations:

- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:44`
- `.github/workflows/slash-command-dispatch.yml:8`
- `.github/workflows/update-major-version.yml:14`

### missing-permissions (severity: medium)

Three workflow files have no top-level 'permissions:' key and no job-level 'permissions:' key on any job, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions. automerge-dependabot.yml, slash-command-dispatch.yml, and update-major-version.yml all lack any permissions declaration.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and 4 workflow files:

1. action.yml (script-injection, static-inline-injection, github-env-injection): Moved all ${{ inputs.* }} and ${{ steps.params.outputs.* }} expressions out of run: blocks into env: blocks. Sanitized comment output with printf/tr before writing to $GITHUB_OUTPUT. Used heredoc delimiter for multi-line comment output. Rewrote 'Close Pull' step to use a bash array (args=()) to build gh pr close arguments safely, keeping --comment and its value as separate array elements.

2. update-major-version.yml (script-injection, missing-permissions, unpinned-uses): Moved ${{ github.event.inputs.main_version }} and ${{ github.event.inputs.target }} to env: blocks. Added 'permissions: contents: write'. Pinned actions/checkout@v3 to full SHA a37ce9120846195fa4ece8f58b268e6043cb2f26.

3. automerge-dependabot.yml (missing-permissions, unpinned-uses): Added 'permissions: pull-requests: write, contents: write'. Pinned peter-evans/enable-pull-request-automerge@v3 to SHA a660677d5469627102a1c1e11409dd063606628d.

4. ci.yml (unpinned-uses): Pinned all 4 unpinned actions to full SHAs (actions/checkout, actions/upload-artifact, peter-evans/create-pull-request, actions/download-artifact).

5. slash-command-dispatch.yml (missing-permissions, unpinned-uses): Added 'permissions: issues: write, pull-requests: write'. Pinned peter-evans/slash-command-dispatch@v3 to SHA f996d7b7aae9059759ac55e978cff76d91853301.

