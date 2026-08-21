<!-- markdownlint-disable -->

# Hardening Report: tj-actions--changed-files/v47

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--changed-files/v47** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings (rule a). In matrix-example.yml, line 40: `echo ${{ matrix.files }}` — the matrix.files value is interpolated directly and unquoted into the shell command, allowing shell metacharacter injection. The value originates from workflow-controllable data (matrix context).

Locations:

- `.github/workflows/matrix-example.yml:40`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings (rule a). In workflow-run-example.yml, lines 25 and 38: `echo "${{ steps.changed-files.outputs.all_changed_files }}"` — the steps output value is interpolated directly into a double-quoted shell string, allowing shell metacharacter injection via attacker-controlled file names.

Locations:

- `.github/workflows/workflow-run-example.yml:25`
- `.github/workflows/workflow-run-example.yml:38`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings (rule a). In issue-comment-job-example.yml, multiple steps use patterns like: `echo "Invalid output: Expected to include ... got (${{ steps.changed-files-all-old-new-renamed-files.outputs.all_old_new_renamed_files }})"` — step outputs (which can contain attacker-controlled file names) are interpolated directly into double-quoted shell strings, enabling command injection.

Locations:

- `.github/workflows/issue-comment-job-example.yml:68`
- `.github/workflows/issue-comment-job-example.yml:74`
- `.github/workflows/issue-comment-job-example.yml:80`
- `.github/workflows/issue-comment-job-example.yml:86`
- `.github/workflows/issue-comment-job-example.yml:130`
- `.github/workflows/issue-comment-job-example.yml:136`
- `.github/workflows/issue-comment-job-example.yml:142`
- `.github/workflows/issue-comment-job-example.yml:148`

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell command strings (rule a) in test.yml. Multiple patterns found: (1) `for file in ${{ steps.changed-files-dir1.outputs.modified_files }}; do` and `for file in ${{ steps.changed-files-dir2.outputs.modified_files }}; do` — step outputs interpolated directly into for-loop word-splitting, allowing shell injection via attacker-controlled filenames. (2) Numerous `echo "${{ toJSON(steps.*.outputs) }}"` patterns — step outputs interpolated into double-quoted shell strings. (3) Multiple `echo "Expected: ... got ${{ steps.*.outcome }}"` and `echo "Invalid output: ... got (${{ steps.*.outputs.* }})"` patterns. All of these allow shell metacharacter injection when file names or outputs contain special characters.

Locations:

- `.github/workflows/test.yml:107`
- `.github/workflows/test.yml:130`
- `.github/workflows/test.yml:356`
- `.github/workflows/test.yml:380`
- `.github/workflows/test.yml:393`
- `.github/workflows/test.yml:416`
- `.github/workflows/test.yml:441`
- `.github/workflows/test.yml:1100`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in four workflow files by moving all ${{ }} expressions from run: shell command strings into env: blocks:

1. **matrix-example.yml** (line 40): Moved `${{ matrix.files }}` to env var `MATRIX_FILES`.

2. **workflow-run-example.yml** (lines 25, 38): Moved `${{ steps.changed-files.outputs.all_changed_files }}` to env var `ALL_CHANGED_FILES` in both on-success and on-failure jobs.

3. **issue-comment-job-example.yml** (lines 68, 74, 80, 86, 130, 136, 142, 148): Fixed 8 injection points in both pr_commented and issue_commented jobs - moved `all_old_new_renamed_files` and `renamed_files` outputs to env vars.

4. **test.yml** (lines 107, 130, 356, 380, 393, 416, 441, 1100 and many more): Fixed all injection patterns throughout the file:
   - Three `for file in ${{ ... }}` loops moved to env vars
   - Multiple `echo "${{ toJSON(...) }}"` patterns moved to env vars
   - Multiple `echo "... got ${{ steps.*.outcome }}"` patterns moved to env vars
   - Multiple `echo "Invalid output: ... got (${{ steps.*.outputs.* }})"` patterns moved to env vars
   - Multiple `if [[ "${{ ... }}" != "false" ]]` patterns moved to env vars
   - Array assignment `=(${{ ... }})` patterns replaced with env var + read -ra
   - All patterns in test-using-since-and-until, test-similar-base-and-commit-sha, test-non-existent-base-sha, test-non-existent-sha, test-dir-names-nested-folder, test-non-existing-repository, test-submodules, test-yaml, test-recover-deleted-file, test-dir-names-deleted-files-include-only-deleted-dirs-single-file, test-dir-names-deleted-files-include-only-deleted-dirs-directory, test-since-last-remote-commit, and test jobs were fixed.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings across 5 workflow files:

1. matrix-example.yml: Moved `${{ steps.changed-files.outputs.all_changed_files }}` to env var ALL_CHANGED_FILES in the 'List all changed files' step.

2. multi-job-example.yml: Fixed 4 instances of `echo '${{ ... }}'` by moving expressions to env vars (ALL_CHANGED_FILES) in all 4 jobs.

3. manual-triggered-job-example.yml: Fixed 3 instances of `echo '${{ toJSON(...) }}'` by moving expressions to env vars (CHANGED_FILES_OUTPUTS, CHANGED_FILES_GLOB_OUTPUTS, CHANGED_FILES_GLOB_ALL_OUTPUTS).

4. issue-comment-job-example.yml: Fixed 6 instances of `echo '${{ toJSON(...) }}'` across both pr_commented and issue_commented jobs by moving expressions to env vars.

5. test.yml: Fixed 4 instances of Rule (a) - moved toJSON expressions for dir1, dir2, changed-files-since, and changed-files-until outputs to env vars. Fixed 3 instances of Rule (b) - replaced unquoted `for file in $MODIFIED_FILES` loops with safe xargs-based iteration using `if [ -n "$MODIFIED_FILES" ]; then while IFS= read -r -d '' file; do ... done < <(printf '%s' "$MODIFIED_FILES" | xargs printf '%s\0'); fi`.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/.github/workflows/test.yml at the 'List all modified files' step. Moved `${{ steps.changed-files-comma.outputs.modified_files }}` from the run: shell command string into an env: block as `MODIFIED_FILES: ${{ steps.changed-files-comma.outputs.modified_files }}`, and updated the here-string to use the plain environment variable `$MODIFIED_FILES` instead of the direct expression interpolation.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 46 occurrences of script injection in .github/workflows/test.yml. Each 'Show output' step that used `echo '${{ toJSON(steps.*.outputs) }}'` was converted to use an `env:` block with `OUTPUTS: ${{ toJSON(...) }}` and the run script now uses `echo "$OUTPUTS"`. This prevents GitHub Actions from interpolating ${{ }} expressions directly into shell commands. Special cases handled: (1) multi-line echo steps for changed-files-json and changed-files-json-unescaped used two separate env vars (OUTPUTS and ALL_CHANGED_FILES); (2) steps with both echo and cat commands kept the cat command unchanged while fixing the echo; (3) steps with conditional `if:` clauses preserved those conditions.

