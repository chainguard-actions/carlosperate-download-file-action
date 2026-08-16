<!-- markdownlint-disable -->

# Hardening Report: carlosperate--download-file-action/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **carlosperate--download-file-action/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `actions/checkout@v3` (a mutable tag reference) in two jobs instead of a pinned full 40-character commit SHA. If the tag is moved or the upstream repository is compromised, the action could execute arbitrary code. Both occurrences must be replaced with a SHA-pinned reference, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3`.

Locations:

- `.github/workflows/action-test.yml:10`
- `.github/workflows/action-test.yml:31`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands (sub-rule a). This allows expression values to be interpreted as shell code before the shell ever sees them. Affected expressions include `${{ steps.download-file-example.outputs.file-path }}`, `${{ steps.download-file-simple.outputs.file-path }}`, `${{ steps.download-file-name.outputs.file-path }}`, `${{ steps.download-file-location.outputs.file-path }}`, `${{ steps.download-file-relative.outputs.file-path }}`, `${{ steps.download-file-absolute.outputs.file-path }}`, `${{ steps.download-file-both.outputs.file-path }}`, and `${{ github.workspace }}`. These must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g. `"$FILE_PATH"`).

Locations:

- `.github/workflows/action-test.yml:22`
- `.github/workflows/action-test.yml:44`
- `.github/workflows/action-test.yml:47`
- `.github/workflows/action-test.yml:49`
- `.github/workflows/action-test.yml:55`
- `.github/workflows/action-test.yml:60`
- `.github/workflows/action-test.yml:63`
- `.github/workflows/action-test.yml:65`
- `.github/workflows/action-test.yml:71`
- `.github/workflows/action-test.yml:76`
- `.github/workflows/action-test.yml:79`
- `.github/workflows/action-test.yml:81`
- `.github/workflows/action-test.yml:87`
- `.github/workflows/action-test.yml:92`
- `.github/workflows/action-test.yml:95`
- `.github/workflows/action-test.yml:97`
- `.github/workflows/action-test.yml:98`
- `.github/workflows/action-test.yml:103`
- `.github/workflows/action-test.yml:108`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and neither of its jobs (`lint-build`, `action-test`) defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal `permissions:` block such as `permissions: {}` or `permissions: contents: read` should be added at the top level or per job.

Locations:

- `.github/workflows/action-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/action-test.yml: (1) Pinned both actions/checkout@v3 references to full SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 with # v3 comment. (2) Moved all ${{ }} expressions from run: blocks into env: blocks, referencing them as double-quoted shell variables ($FILE_PATH, $WORKSPACE); Python inline string interpolation was converted to use os.environ[] lookups instead. (3) Added top-level 'permissions: {}' to enforce least-privilege token access.

