<!-- markdownlint-disable -->

# Hardening Report: tj-actions--changed-files/v47.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--changed-files/v47.0.6** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps interpolate ${{ }} expressions directly inside shell commands. GitHub Actions resolves these expressions before the shell sees them, so even single-quoted strings like echo '${{ steps.changed-files.outputs.all_changed_files }}' are vulnerable. Affected steps include:
- Line 27: `run: echo '${{ steps.changed-files.outputs.all_changed_files }}'`
- Line 38: `run: |\n  echo ${{ matrix.files }}` — also sub-rule (b): unquoted shell expansion of a workflow-controlled value (matrix.files is derived from changed file names, which can be attacker-controlled via PR).

Locations:

- `.github/workflows/matrix-example.yml:27`
- `.github/workflows/matrix-example.yml:38`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps interpolate ${{ }} expressions directly inside shell commands. Affected steps include:
- Line 28: `run: echo '${{ steps.changed-files.outputs.all_changed_files }}'`
- Line 34: `run: |\n  echo '${{ needs.changed-files.outputs.all_changed_files }}'`
- Line 50: `run: echo '${{ steps.changed-files.outputs.all_changed_files }}'`
- Line 57: `run: |\n  echo '${{ needs.changed-files-rest-api.outputs.all_changed_files }}'`
Single-quoting does not protect against ${{ }} interpolation since GitHub Actions resolves expressions before the shell executes the command.

Locations:

- `.github/workflows/multi-job-example.yml:28`
- `.github/workflows/multi-job-example.yml:34`
- `.github/workflows/multi-job-example.yml:50`
- `.github/workflows/multi-job-example.yml:57`

### script-injection (severity: high)

Sub-rule (a): run: steps interpolate ${{ steps.changed-files.outputs.all_changed_files }} directly inside shell echo commands. The action output can contain attacker-controlled filenames (e.g. from a PR adding a file with a malicious name). Affected steps:
- Line 23: `echo "${{ steps.changed-files.outputs.all_changed_files }}"`
- Line 33: `echo "${{ steps.changed-files.outputs.all_changed_files }}"`

Locations:

- `.github/workflows/workflow-run-example.yml:23`
- `.github/workflows/workflow-run-example.yml:33`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps interpolate ${{ }} expressions directly inside shell commands. Affected steps include echo lines with ${{ toJSON(steps.changed-files.outputs) }}, ${{ steps.changed-files-all-old-new-renamed-files.outputs.all_old_new_renamed_files }}, and ${{ steps.changed-files-all-old-new-renamed-files.outputs.renamed_files }} embedded in double-quoted echo strings inside run: blocks. These outputs can contain attacker-controlled filenames from pull requests.

Locations:

- `.github/workflows/issue-comment-job-example.yml:33`
- `.github/workflows/issue-comment-job-example.yml:47`
- `.github/workflows/issue-comment-job-example.yml:53`
- `.github/workflows/issue-comment-job-example.yml:63`
- `.github/workflows/issue-comment-job-example.yml:70`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps interpolate ${{ }} expressions directly inside shell commands. Affected steps include echo lines with ${{ toJSON(steps.changed-files.outputs) }}, ${{ toJSON(steps.changed-files-glob.outputs) }}, and ${{ toJSON(steps.changed-files-glob-all-old-new-renamed-files.outputs) }} embedded in single-quoted echo strings inside run: blocks. Single-quoting does not prevent GitHub Actions from interpolating ${{ }} before the shell executes the command.

Locations:

- `.github/workflows/manual-triggered-job-example.yml:33`
- `.github/workflows/manual-triggered-job-example.yml:43`
- `.github/workflows/manual-triggered-job-example.yml:53`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps across this large workflow file interpolate ${{ }} expressions directly inside shell commands. Notable violations include:
- `for file in ${{ steps.changed-files-dir1.outputs.modified_files }}; do` — expression interpolated directly into a for-loop word-split context
- `for file in ${{ steps.changed-files-dir2.outputs.modified_files }}; do` — same pattern
- `for file in ${{ steps.changed-files.outputs.modified_files }}; do` — same pattern
- `echo "Your README.md has been modified ${{ steps.changed-files.outputs.modified_files }}."` — expression in double-quoted echo
- Numerous `echo '${{ toJSON(...) }}'` steps throughout the file
All action outputs can contain attacker-controlled filenames from pull requests.

Locations:

- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:127`
- `.github/workflows/test.yml:1197`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities across 6 workflow files by moving every ${{ }} expression from run: shell commands into the step's env: block and referencing them as plain environment variables. Key fixes: (1) matrix-example.yml: fixed echo with all_changed_files output and unquoted for-loop with matrix.files; (2) multi-job-example.yml: fixed 4 echo patterns with step/job outputs; (3) workflow-run-example.yml: fixed 2 echo patterns with all_changed_files; (4) issue-comment-job-example.yml: rewrote file fixing all toJSON echo patterns and error message patterns with renamed_files/all_old_new_renamed_files outputs; (5) manual-triggered-job-example.yml: fixed 3 toJSON echo patterns; (6) test.yml: fixed 50+ occurrences including for-loop patterns (for file in ${{ }}), IFS/read patterns, if [[ "${{ }}" ]] patterns, array expansion patterns (${{ }}), and numerous echo '${{ toJSON(...) }}' patterns throughout the large CI workflow file.

