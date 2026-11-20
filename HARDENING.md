<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Set Parameters' step puts expressions straight into its run: script: `if [ -n "${{ inputs.comment }}" ]; then`, `echo comment="--comment \"${{ inputs.comment }}\"" >> $GITHUB_OUTPUT` and `if [ "${{ inputs.delete-branch }}" = true ]; then`. A comment containing quotes or $(...) can run arbitrary commands.

Locations:

- `action.yml:25`

### script-injection (severity: high)

Rule (a): the 'Close Pull' step puts expressions straight into its run: script: `gh pr close -R "${{ inputs.repository }}"`, `${{ steps.params.outputs.comment }}` (unquoted, and it carries the user's comment), `${{ steps.params.outputs.delete-branch }}` and `"${{ inputs.pull-request-number }}"`. These are substituted into the script before the shell runs, so they allow command injection.

Locations:

- `action.yml:34`

### github-env-injection (severity: high)

The 'Set Parameters' step writes the untrusted inputs.comment to $GITHUB_OUTPUT (`echo comment="--comment \"${{ inputs.comment }}\"" >> $GITHUB_OUTPUT`) without first stripping newlines with `printf '%s' ... | tr -d '\n\r'`. A comment with newlines can add extra step outputs, such as overriding delete-branch.

Locations:

- `action.yml:26`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index db72b79..62d524d 100644
--- a/action.yml
+++ b/action.yml
@@ -18,26 +18,54 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Set Parameters
       id: params
       shell: bash
+      env:
+        INPUT_COMMENT: ${{ inputs.comment }}
+        INPUT_DELETE_BRANCH: ${{ inputs.delete-branch }}
       run: |
-        if [ -n "${{ inputs.comment }}" ]; then
-          echo comment="--comment \"${{ inputs.comment }}\"" >> $GITHUB_OUTPUT
+        if [ -n "$INPUT_COMMENT" ]; then
+          safe_comment=$(printf '%s' "$INPUT_COMMENT" | tr -d '\n\r')
+          echo "comment=$safe_comment" >> "$GITHUB_OUTPUT"
         fi
-        if [ "${{ inputs.delete-branch }}" = true ]; then
-          echo delete-branch="--delete-branch" >> $GITHUB_OUTPUT
+        if [ "$INPUT_DELETE_BRANCH" = true ]; then
+          echo "delete-branch=true" >> "$GITHUB_OUTPUT"
         fi
-
     - name: Close Pull
       shell: bash
-      run: |
-        gh pr close -R "${{ inputs.repository }}" \
-        ${{ steps.params.outputs.comment }} \
-        ${{ steps.params.outputs.delete-branch }} \
-        "${{ inputs.pull-request-number }}"
       env:
         GH_TOKEN: ${{ inputs.token }}
+        INPUT_REPOSITORY: ${{ inputs.repository }}
+        INPUT_PR_NUMBER: ${{ inputs.pull-request-number }}
+        PARAM_COMMENT: ${{ steps.params.outputs.comment }}
+        PARAM_DELETE_BRANCH: ${{ steps.params.outputs.delete-branch }}
+      run: |
+        args=()
+        if [ -n "$PARAM_COMMENT" ]; then
+          args+=(--comment "$PARAM_COMMENT")
+        fi
+        if [ "$PARAM_DELETE_BRANCH" = true ]; then
+          args+=(--delete-branch)
+        fi
+        gh pr close -R "$INPUT_REPOSITORY" "${args[@]}" "$INPUT_PR_NUMBER"
 branding:
   icon: 'git-pull-request'
   color: 'gray-dark'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

action.yml: every ${{ }} expression in both run: scripts is now passed through the step's env: block. The 'Set Parameters' step strips newlines from the comment before writing it to GITHUB_OUTPUT. It now outputs plain values: the comment text, and delete-branch=true. It no longer outputs ready-made flag strings. The 'Close Pull' step builds the gh pr close arguments in a bash array, so --comment and its value stay separate arguments and every value is double-quoted. This re-applies the previous harden's patch. Behaviour change: a multi-line comment is joined into one line (upstream's echo to GITHUB_OUTPUT could not handle multi-line comments correctly either). Not run or tested here.

