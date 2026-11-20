<!-- markdownlint-disable -->

# Hardening Report: peter-evans--close-pull/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--close-pull/v3.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Set Parameters' step puts expressions straight into the run: script. Lines: `if [ -n "${{ inputs.comment }}" ]; then`, `comment="--comment \"${{ inputs.comment }}\""` and `if [ "${{ inputs.delete-branch }}" = true ]; then`. A comment input that contains quotes or $(...) is run as shell code. Fix: pass the inputs through env: variables and expand them with double quotes.

Locations:

- `action.yml:25`
- `action.yml:26`
- `action.yml:32`

### script-injection (severity: high)

Rule (a): the 'Close Pull' step puts expressions straight into the run: script. Lines: `gh pr close -R "${{ inputs.repository }}" \`, `${{ steps.params.outputs.comment }} \` (unquoted, and it carries the user-supplied comment text), `${{ steps.params.outputs.delete-branch }} \` and `"${{ inputs.pull-request-number }}"`. Fix: pass these values through env: variables, expand them with double quotes, and build the optional arguments with the guarded form or a bash array.

Locations:

- `action.yml:39`
- `action.yml:40`
- `action.yml:41`
- `action.yml:42`

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

### github-env-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. In the 'Set Parameters' step, the untrusted value `${{ inputs.comment }}` is interpolated directly into the shell variable `comment` (line 24), which is then written to $GITHUB_OUTPUT via `echo "$comment" >> $GITHUB_OUTPUT` (line 27) without any sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled comment containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, poisoning subsequent steps.

Locations:

- `action.yml:27`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 454994b..5cbbd61 100644
--- a/action.yml
+++ b/action.yml
@@ -18,30 +18,58 @@ inputs:
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
-          comment="--comment \"${{ inputs.comment }}\""
+        if [ -n "$INPUT_COMMENT" ]; then
           delimiter="$(openssl rand -hex 8)"
           echo "comment<<$delimiter" >> $GITHUB_OUTPUT
-          echo "$comment" >> $GITHUB_OUTPUT
+          echo "$INPUT_COMMENT" >> $GITHUB_OUTPUT
           echo "$delimiter" >> $GITHUB_OUTPUT
         fi
-        if [ "${{ inputs.delete-branch }}" = true ]; then
-          echo delete-branch="--delete-branch" >> $GITHUB_OUTPUT
+        if [ "$INPUT_DELETE_BRANCH" = true ]; then
+          echo "delete-branch=true" >> $GITHUB_OUTPUT
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
+        INPUT_PULL_REQUEST_NUMBER: ${{ inputs.pull-request-number }}
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
+        gh pr close -R "$INPUT_REPOSITORY" \
+          "${args[@]}" \
+          "$INPUT_PULL_REQUEST_NUMBER"
 branding:
   icon: 'git-pull-request'
   color: 'gray-dark'

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

action.yml: moved all ${{ }} expressions (inputs.comment, delete-branch, repository, pull-request-number, step outputs) into env: blocks. Set Parameters writes the raw comment to GITHUB_OUTPUT via a heredoc block with a random openssl delimiter (multi-line preserved, no key injection) and delete-branch=true as a flag. Close Pull builds optional args with a bash array (--comment "$PARAM_COMMENT", --delete-branch) and double-quotes repository and PR number. Re-applies the previous harden's changes.

