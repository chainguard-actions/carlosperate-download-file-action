<!-- markdownlint-disable -->

# Hardening Report: carlosperate--download-file-action/v1.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **carlosperate--download-file-action/v1.1.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v3` (a mutable tag reference) in two steps instead of a pinned full 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved or the upstream action is compromised. Both occurrences are in `.github/workflows/action-test.yml` — one in the `lint-build` job (line 10) and one in the `action-test` job (line 23).

Locations:

- `.github/workflows/action-test.yml:10`
- `.github/workflows/action-test.yml:23`

### permissions (severity: medium)

missing-permissions: The workflow file `.github/workflows/action-test.yml` has no top-level `permissions:` key, and neither the `lint-build` job nor the `action-test` job defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., write access to contents and packages).

Locations:

- `.github/workflows/action-test.yml:1`

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` steps in the `action-test` job directly interpolate `${{ ... }}` expressions inside shell commands. The values `${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}` are substituted by the Actions template engine before the shell ever sees the command, allowing an attacker who controls the step output or workspace path to inject arbitrary shell commands. Several steps also use the interpolated value completely unquoted (e.g., `rm ${{ steps.download-poetry-simple.outputs.file-path }}`), compounding the risk. Affected lines include: 38 (`echo ... ${{ steps.download-poetry-simple.outputs.file-path }}`), 40 (`python -c ... '${{ steps.download-poetry-simple.outputs.file-path }}'`), 41 (`rm ${{ steps.download-poetry-simple.outputs.file-path }}`), 47 (`echo ... ${{ steps.download-poetry-name.outputs.file-path }}`), 49 (`python -c ... '${{ steps.download-poetry-name.outputs.file-path }}'`), 50 (`rm ${{ steps.download-poetry-name.outputs.file-path }}`), 57 (`echo ... ${{ steps.download-poetry-location.outputs.file-path }}`), 59 (`python -c ... '${{ steps.download-poetry-location.outputs.file-path }}'`), 65 (`echo ... ${{ steps.download-poetry-relative.outputs.file-path }}`), 67 (`python -c ... '${{ steps.download-poetry-relative.outputs.file-path }}'`), 68 (`rm ${{ steps.download-poetry-relative.outputs.file-path }}`), 76 (`echo ... ${{ steps.download-poetry-absolute.outputs.file-path }}`), 77 (`python -c ... r'${{ github.workspace }}/a_folder/...'`), 78 (`python -c ... r'${{ steps.download-poetry-absolute.outputs.file-path }}'`), 79 (`ls ${{ github.workspace }}/a_folder && rm -r ${{ github.workspace }}/a_folder`), and 87 (`echo ... ${{ steps.download-poetry-both.outputs.file-path }}`).

Locations:

- `.github/workflows/action-test.yml:38`
- `.github/workflows/action-test.yml:40`
- `.github/workflows/action-test.yml:41`
- `.github/workflows/action-test.yml:47`
- `.github/workflows/action-test.yml:49`
- `.github/workflows/action-test.yml:50`
- `.github/workflows/action-test.yml:57`
- `.github/workflows/action-test.yml:59`
- `.github/workflows/action-test.yml:65`
- `.github/workflows/action-test.yml:67`
- `.github/workflows/action-test.yml:68`
- `.github/workflows/action-test.yml:76`
- `.github/workflows/action-test.yml:77`
- `.github/workflows/action-test.yml:78`
- `.github/workflows/action-test.yml:79`
- `.github/workflows/action-test.yml:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/action-test.yml: (1) Pinned both actions/checkout@v3 references to full SHA f43a0e5ff2bd294095638e18286ca9a3d1956744 with # v3 comment. (2) Added top-level 'permissions: {}' block to restrict default GITHUB_TOKEN permissions. (3) Moved all 16 ${{ }} expressions from run: shell commands into env: blocks — step outputs use FILE_PATH env var (referenced as "$FILE_PATH" in shell or os.environ['FILE_PATH'] in Python), and github.workspace uses GH_WORKSPACE env var (referenced as "$GH_WORKSPACE" in shell or os.environ['GH_WORKSPACE'] in Python). All shell variable expansions are properly double-quoted.

