<!-- markdownlint-disable -->

# Hardening Report: carlosperate--download-file-action/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **carlosperate--download-file-action/v2.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v6` (a mutable tag reference, not a pinned 40-character commit SHA) in two steps. If the tag is moved or the action is compromised, the workflow will silently execute different code. Pin to a full SHA, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/action-test.yml:10`
- `.github/workflows/action-test.yml:32`

### script-injection (severity: high)

Multiple `run:` blocks in the workflow directly interpolate GitHub Actions expressions (`${{ steps.*.outputs.file-path }}`, `${{ github.workspace }}`) into shell commands via YAML template substitution before the shell ever sees the value. An attacker who can influence the downloaded file path (e.g. via a crafted server response that sets the output) could inject arbitrary shell metacharacters. Sub-rule (a): direct `${{ ... }}` interpolation inside `run:` strings. Affected lines include: line 21 (`rm ${{ steps.download-file-example.outputs.file-path }}`), line 47 (`echo ... ${{ steps.download-file-simple.outputs.file-path }}`), line 51 (`python -c ... r'${{ steps.download-file-simple.outputs.file-path }}'`), line 52 (`rm ${{ steps.download-file-simple.outputs.file-path }}`), line 60 (`echo ... ${{ steps.download-file-name.outputs.file-path }}`), line 64 (`python -c ... ${{ steps.download-file-name.outputs.file-path }}`), line 65 (`rm ${{ steps.download-file-name.outputs.file-path }}`), line 73 (`echo ... ${{ steps.download-file-location.outputs.file-path }}`), line 77 (`python -c ... ${{ steps.download-file-location.outputs.file-path }}`), line 86 (`echo ... ${{ steps.download-file-relative.outputs.file-path }}`), line 90 (`python -c ... ${{ steps.download-file-relative.outputs.file-path }}`), line 91 (`rm ${{ steps.download-file-relative.outputs.file-path }}`), line 99 (`echo ... ${{ steps.download-file-absolute.outputs.file-path }}`), line 101 (`python -c ... ${{ github.workspace }}`), line 103 (`python -c ... ${{ steps.download-file-absolute.outputs.file-path }} ... ${{ github.workspace }}`), line 104 (`ls ${{ github.workspace }} ...`), line 113 (`echo ... ${{ steps.download-file-both.outputs.file-path }}`), line 117 (`python -c ... ${{ steps.download-file-both.outputs.file-path }}`), line 155 (`echo ... ${{ steps.download-extensionless.outputs.file-path }}`), line 157 (`python -c ... ${{ steps.download-extensionless.outputs.file-path }}`), line 158 (`rm ${{ steps.download-extensionless.outputs.file-path }}`).

Locations:

- `.github/workflows/action-test.yml:21`
- `.github/workflows/action-test.yml:47`
- `.github/workflows/action-test.yml:51`
- `.github/workflows/action-test.yml:52`
- `.github/workflows/action-test.yml:60`
- `.github/workflows/action-test.yml:64`
- `.github/workflows/action-test.yml:65`
- `.github/workflows/action-test.yml:73`
- `.github/workflows/action-test.yml:77`
- `.github/workflows/action-test.yml:86`
- `.github/workflows/action-test.yml:90`
- `.github/workflows/action-test.yml:91`
- `.github/workflows/action-test.yml:99`
- `.github/workflows/action-test.yml:101`
- `.github/workflows/action-test.yml:103`
- `.github/workflows/action-test.yml:104`
- `.github/workflows/action-test.yml:113`
- `.github/workflows/action-test.yml:117`
- `.github/workflows/action-test.yml:155`
- `.github/workflows/action-test.yml:157`
- `.github/workflows/action-test.yml:158`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and neither the `lint-build` job nor the `action-test` job defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary (e.g. write access to contents, packages, etc.).

Locations:

- `.github/workflows/action-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/action-test.yml: (1) Pinned both `actions/checkout@v6` references to full SHA `d23441a48e516b6c34aea4fa41551a30e30af803 # v6`. (2) Moved all 21 `${{ steps.*.outputs.file-path }}` and `${{ github.workspace }}` expressions out of `run:` blocks into `env:` blocks, referencing them as `$FILE_PATH` and `$WORKSPACE` in shell, and `os.environ['FILE_PATH']`/`os.environ['WORKSPACE']` in Python inline scripts. (3) Added `permissions: {}` at the top level to enforce least-privilege token access.

