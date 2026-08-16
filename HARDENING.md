<!-- markdownlint-disable -->

# Hardening Report: carlosperate--download-file-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **carlosperate--download-file-action/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v3` (a mutable tag reference) in two steps. These should be pinned to a full 40-character commit SHA (e.g., `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`) to prevent supply-chain attacks via tag mutation.

Locations:

- `.github/workflows/action-test.yml:10`
- `.github/workflows/action-test.yml:24`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and neither job (`lint-build` nor `action-test`) defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block (e.g., `permissions: contents: read`) should be added.

Locations:

- `.github/workflows/action-test.yml:1`

### script-injection (severity: high)

Multiple `run:` steps directly interpolate GitHub Actions expressions (`${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}`) inside shell commands — sub-rule (a) violation. These values flow through YAML template substitution before the shell sees them, allowing an attacker who controls the output (e.g., via a crafted file-url) to inject shell metacharacters. Affected lines include:
- Line 38: `run: echo "The file was downloaded to ${{ steps.download-poetry-simple.outputs.file-path }}"`
- Line 41: `run: python -c "... r'${{ steps.download-poetry-simple.outputs.file-path }}' ..."`
- Line 42: `run: ls && rm ${{ steps.download-poetry-simple.outputs.file-path }} && ls`
- Line 50: `run: echo "... ${{ steps.download-poetry-name.outputs.file-path }}"`
- Line 53: `run: python -c "... r'${{ steps.download-poetry-name.outputs.file-path }}' ..."`
- Line 54: `run: ls && rm ${{ steps.download-poetry-name.outputs.file-path }} && ls`
- Line 62: `run: echo "... ${{ steps.download-poetry-location.outputs.file-path }}"`
- Line 65: `run: python -c "... r'${{ steps.download-poetry-location.outputs.file-path }}' ..."`
- Line 73: `run: echo "... ${{ steps.download-poetry-relative.outputs.file-path }}"`
- Line 76: `run: python -c "... r'${{ steps.download-poetry-relative.outputs.file-path }}' ..."`
- Line 77: `run: ls ../ && rm ${{ steps.download-poetry-relative.outputs.file-path }} && ls ../`
- Line 85: `run: echo "... ${{ steps.download-poetry-absolute.outputs.file-path }}"`
- Line 87: `run: python -c "... r'${{ github.workspace }}/a_folder/get-poetry.py' ..."`
- Line 88: `run: python -c "... r'${{ steps.download-poetry-absolute.outputs.file-path }}' ..."`
- Line 89: `run: ls ${{ github.workspace }} && ls ${{ github.workspace }}/a_folder && rm -r ${{ github.workspace }}/a_folder && ls ${{ github.workspace }}`
- Line 101: `run: echo "... ${{ steps.download-poetry-both.outputs.file-path }}"`
- Line 104: `run: python -c "... r'${{ steps.download-poetry-both.outputs.file-path }}' ..."`
Fix: move expressions into `env:` variables and reference them as quoted shell variables (e.g., `"$FILE_PATH"`).

Locations:

- `.github/workflows/action-test.yml:38`
- `.github/workflows/action-test.yml:41`
- `.github/workflows/action-test.yml:42`
- `.github/workflows/action-test.yml:50`
- `.github/workflows/action-test.yml:53`
- `.github/workflows/action-test.yml:54`
- `.github/workflows/action-test.yml:62`
- `.github/workflows/action-test.yml:65`
- `.github/workflows/action-test.yml:73`
- `.github/workflows/action-test.yml:76`
- `.github/workflows/action-test.yml:77`
- `.github/workflows/action-test.yml:85`
- `.github/workflows/action-test.yml:87`
- `.github/workflows/action-test.yml:88`
- `.github/workflows/action-test.yml:89`
- `.github/workflows/action-test.yml:101`
- `.github/workflows/action-test.yml:104`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/action-test.yml:
1. unpinned-uses: Pinned both `actions/checkout@v3` references to full SHA `a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3`.
2. missing-permissions: Added top-level `permissions: contents: read` block.
3. script-injection: Moved all 17 `${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}` expressions out of `run:` shell strings into `env:` blocks (as FILE_PATH and WORKSPACE variables), then referenced them as quoted shell variables (`"$FILE_PATH"`, `"$WORKSPACE"`) in the shell commands.

