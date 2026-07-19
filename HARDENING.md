<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml — 'Set Parameters' step (sub-rule a): `${{ inputs.comment }}` and `${{ inputs.delete-branch }}` are interpolated directly inside the `run:` shell script. An attacker-controlled input value is expanded by the YAML template engine before the shell ever sees it, enabling command injection. Offending lines: `if [ -n "${{ inputs.comment }}" ]; then` and `echo comment="--comment \"${{ inputs.comment }}\"" >> $GITHUB_OUTPUT` and `if [ "${{ inputs.delete-branch }}" = true ]; then`.

'Close Pull' step (sub-rule a): `${{ inputs.repository }}`, `${{ steps.params.outputs.comment }}`, `${{ steps.params.outputs.delete-branch }}`, and `${{ inputs.pull-request-number }}` are all interpolated directly inside the `run:` shell command. Offending lines: `gh pr close -R "${{ inputs.repository }}" \`, `${{ steps.params.outputs.comment }} \`, `${{ steps.params.outputs.delete-branch }} \`, `"${{ inputs.pull-request-number }}"`.

Locations:

- `action.yml:23`
- `action.yml:24`
- `action.yml:26`
- `action.yml:33`
- `action.yml:34`
- `action.yml:35`
- `action.yml:36`

### script-injection (severity: high)

update-major-version.yml — 'Tag new target' and 'Push new tag' steps (sub-rule a): `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` are interpolated directly inside `run:` shell commands. These are workflow_dispatch inputs that can be supplied by any user with dispatch access, enabling arbitrary git tag injection or command injection. Offending lines: `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}` and `run: git push origin ${{ github.event.inputs.main_version }} --force`.

Locations:

- `.github/workflows/update-major-version.yml:22`
- `.github/workflows/update-major-version.yml:24`

### github-env-injection (severity: high)

action.yml — 'Set Parameters' step writes `${{ inputs.comment }}` (an attacker-controlled input) directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline embedded in `inputs.comment` can inject additional key=value pairs into GITHUB_OUTPUT, poisoning downstream step outputs. Offending line: `echo comment="--comment \"${{ inputs.comment }}\"" >> $GITHUB_OUTPUT`.

Locations:

- `action.yml:24`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

automerge-dependabot.yml: `uses: peter-evans/enable-pull-request-automerge@v3`

ci.yml: `uses: actions/checkout@v3`, `uses: actions/upload-artifact@v3`, `uses: actions/checkout@v3` (second occurrence), `uses: actions/download-artifact@v3`, `uses: peter-evans/create-pull-request@v5`

slash-command-dispatch.yml: `uses: peter-evans/slash-command-dispatch@v3`

update-major-version.yml: `uses: actions/checkout@v3`

Locations:

- `.github/workflows/automerge-dependabot.yml:8`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:28`
- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:40`
- `.github/workflows/slash-command-dispatch.yml:7`
- `.github/workflows/update-major-version.yml:16`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions.

- automerge-dependabot.yml: no permissions block at top-level or job level.
- slash-command-dispatch.yml: no permissions block at top-level or job level.
- update-major-version.yml: no permissions block at top-level or job level.

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

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repository }}" appears directly in run: block of step "Close Pull"; move to env: map

Locations:

- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pull-request-number }}" appears directly in run: block of step "Close Pull"; move to env: map

Locations:

- `action.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml and workflow files:

1. action.yml - Set Parameters step: moved inputs.comment and inputs.delete-branch to env: block; sanitized comment with tr -d '\n\r' before writing to GITHUB_OUTPUT to prevent env injection.

2. action.yml - Close Pull step: moved all ${{ }} expressions (inputs.repository, inputs.pull-request-number, steps.params.outputs.*) to env: block; used bash array to build optional --comment and --delete-branch arguments safely.

3. update-major-version.yml - Tag new target and Push new tag steps: moved github.event.inputs.main_version and github.event.inputs.target to env: blocks.

4. Pinned all unpinned action references to full commit SHAs: actions/checkout@v3, actions/upload-artifact@v3, actions/download-artifact@v3, peter-evans/create-pull-request@v5, peter-evans/enable-pull-request-automerge@v3, peter-evans/slash-command-dispatch@v3.

5. Added permissions blocks to automerge-dependabot.yml (pull-requests: write), slash-command-dispatch.yml (issues: read, pull-requests: read), and update-major-version.yml (contents: write).

