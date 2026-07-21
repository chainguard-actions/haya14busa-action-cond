<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate ${{ ... }} expressions into shell commands (sub-rule a). The expressions ${{ steps.testval.outputs.value }}, ${{ steps.falseval.outputs.value }}, and ${{ steps.event.outputs.value }} are interpolated directly into shell strings before the shell executes them. Even though these come from steps.*.outputs.*, they are workflow-controllable and flow through YAML template substitution before the shell sees them, enabling script injection. Offending lines:
- Line 28: run: test "${{ steps.testval.outputs.value }}" = "true value"
- Line 36: run: test "${{ steps.falseval.outputs.value }}" = "false value"
- Line 48: run: echo "${{ steps.event.outputs.value }}"
- Line 50: run: test "${{ steps.event.outputs.value }}" = "value for pull request event"
- Line 52: run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"

Locations:

- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:50`
- `.github/workflows/test.yml:52`

### unpinned-uses (severity: high)

All uses: references across all three workflow files use mutable tag refs instead of pinned 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tags are moved or compromised.

release.yml:
- actions/checkout@v2 (line 13)
- haya14busa/action-bumpr@v1 (line 19)
- haya14busa/action-update-semver@v1 (line 23)
- haya14busa/action-cond@v1 (line 29)

reviewdog.yml:
- actions/checkout@v1 (line 11)
- reviewdog/action-eslint@v1 (line 16)

test.yml:
- actions/checkout@v2 (line 12)
- actions/checkout@v2 (line 19)

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:29`
- `.github/workflows/reviewdog.yml:11`
- `.github/workflows/reviewdog.yml:16`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:19`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level permissions: key, and no individual job within any of these files defines a job-level permissions: key. Without explicit permissions, workflows inherit the default repository permissions (which may be broad), violating the principle of least privilege. Affected files: release.yml, reviewdog.yml, test.yml.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across test.yml, release.yml, and reviewdog.yml:

1. script-injection: Moved all 5 ${{ steps.*.outputs.value }} expressions from run: shell strings into env: blocks in test.yml. Each step now references plain environment variables ($TESTVAL, $FALSEVAL, $EVENT_VALUE) in the shell commands.

2. unpinned-uses: Pinned all 8 uses: references to full 40-character commit SHAs (resolved via lookup_action_sha), preserving original tags as comments. Actions pinned: actions/checkout@v2, actions/checkout@v1, haya14busa/action-bumpr@v1, haya14busa/action-update-semver@v1, haya14busa/action-cond@v1, reviewdog/action-eslint@v1.

3. missing-permissions: Added top-level permissions blocks to all three files with minimal required permissions: test.yml gets contents:read; release.yml gets contents:write (needed for gh release create); reviewdog.yml gets contents:read + pull-requests:write + checks:write (needed for reviewdog PR review and check annotations).

