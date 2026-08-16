<!-- markdownlint-disable -->

# Hardening Report: carlosperate--download-file-action/v2.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **carlosperate--download-file-action/v2.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v4` (a mutable tag reference) in two places. These should be pinned to a full 40-character commit SHA to prevent supply-chain attacks via tag mutation. Failing references: `actions/checkout@v4` at lines 10 and 31.

Locations:

- `.github/workflows/action-test.yml:10`
- `.github/workflows/action-test.yml:31`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and neither the `lint-build` job nor the `action-test` job defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`). A minimal permissions block (e.g. `permissions: read-all` or specific scopes) should be added.

Locations:

- `.github/workflows/action-test.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in the workflow directly interpolate `${{ ... }}` expressions inside shell commands (sub-rule a). This causes GitHub Actions to substitute the value into the shell script before the shell parses it, enabling script injection if the value contains shell metacharacters. Affected expressions include `${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}`. All such values should be passed via `env:` variables and then referenced as quoted shell variables (e.g. `"$VAR"`) instead. Offending lines include:
- Line 22: `run: rm ${{ steps.download-file-example.outputs.file-path }}`
- Line 44: `run: echo "The file was downloaded to ${{ steps.download-file-simple.outputs.file-path }}"`
- Line 47: `run: python -c "...r'${{ steps.download-file-simple.outputs.file-path }}'..."`
- Line 48: `run: ls && rm ${{ steps.download-file-simple.outputs.file-path }} && ls`
- Line 55: `run: echo "The file was downloaded to ${{ steps.download-file-name.outputs.file-path }}"`
- Line 58: `run: python -c "...r'${{ steps.download-file-name.outputs.file-path }}'..."`
- Line 59: `run: ls && rm ${{ steps.download-file-name.outputs.file-path }} && ls`
- Line 66: `run: echo "The file was downloaded to ${{ steps.download-file-location.outputs.file-path }}"`
- Line 69: `run: python -c "...r'${{ steps.download-file-location.outputs.file-path }}'..."`
- Line 76: `run: echo "The file was downloaded to ${{ steps.download-file-relative.outputs.file-path }}"`
- Line 79: `run: python -c "...r'${{ steps.download-file-relative.outputs.file-path }}'..."`
- Line 80: `run: ls ../ && rm ${{ steps.download-file-relative.outputs.file-path }} && ls ../`
- Line 86: `run: echo "The file was downloaded to ${{ steps.download-file-absolute.outputs.file-path }}"`
- Line 87: `run: python -c "...r'${{ github.workspace }}/a_folder/..."`
- Line 88: `run: python -c "...r'${{ steps.download-file-absolute.outputs.file-path }}'...r'${{ github.workspace }}/..."`
- Line 89: `run: ls ${{ github.workspace }} && ls ${{ github.workspace }}/a_folder && ...`
- Line 96: `run: echo "The file was downloaded to ${{ steps.download-file-both.outputs.file-path }}"`
- Line 99: `run: python -c "...r'${{ steps.download-file-both.outputs.file-path }}'..."`

Locations:

- `.github/workflows/action-test.yml:22`
- `.github/workflows/action-test.yml:44`
- `.github/workflows/action-test.yml:47`
- `.github/workflows/action-test.yml:48`
- `.github/workflows/action-test.yml:55`
- `.github/workflows/action-test.yml:58`
- `.github/workflows/action-test.yml:59`
- `.github/workflows/action-test.yml:66`
- `.github/workflows/action-test.yml:69`
- `.github/workflows/action-test.yml:76`
- `.github/workflows/action-test.yml:79`
- `.github/workflows/action-test.yml:80`
- `.github/workflows/action-test.yml:86`
- `.github/workflows/action-test.yml:87`
- `.github/workflows/action-test.yml:88`
- `.github/workflows/action-test.yml:89`
- `.github/workflows/action-test.yml:96`
- `.github/workflows/action-test.yml:99`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/action-test.yml: (1) Pinned both `actions/checkout@v4` references to full SHA `11d5960a326750d5838078e36cf38b85af677262 # v4`. (2) Added top-level `permissions: contents: read` block. (3) Moved all `${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}` expressions out of `run:` shell strings into `env:` blocks, referencing them as quoted shell variables (`"$FILE_PATH"`, `"$WORKSPACE"`) or via `os.environ` in Python one-liners.

