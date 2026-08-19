<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tag refs instead of pinned 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if those tags are moved or compromised.

Failing references:
- release.yml: actions/checkout@v2, haya14busa/action-bumpr@v1, haya14busa/action-update-semver@v1, haya14busa/action-cond@v1, actions/create-release@v1
- reviewdog.yml: actions/checkout@v1, reviewdog/action-eslint@v1
- test.yml: actions/checkout@v2 (×2)

Locations:

- `.github/workflows/release.yml:12`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:38`
- `.github/workflows/reviewdog.yml:10`
- `.github/workflows/reviewdog.yml:16`
- `.github/workflows/test.yml:9`
- `.github/workflows/test.yml:17`

### script-injection (severity: high)

test.yml contains multiple `run:` steps that directly interpolate `${{ steps.*.outputs.value }}` expressions into shell command strings (sub-rule a). The value produced by the action under test is injected verbatim into the shell before quoting can occur, allowing an attacker who can influence the action's output to inject arbitrary shell commands.

Offending lines:
- `run: test "${{ steps.testval.outputs.value }}" = "true value"`
- `run: test "${{ steps.falseval.outputs.value }}" = "false value"`
- `run: echo "${{ steps.event.outputs.value }}"`
- `run: test "${{ steps.event.outputs.value }}" = "value for pull request event"`
- `run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"`

These should be moved to env: variables and referenced as shell variables (e.g. `"$STEP_OUTPUT"`) instead.

Locations:

- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:51`
- `.github/workflows/test.yml:54`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the default token permissions (which may be read/write depending on repository settings), granting broader access than necessary.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across the three workflow files:

1. unpinned-uses: Pinned all 9 mutable tag references to full 40-char commit SHAs (with original tag preserved in comments): actions/checkout@v2→ee0669bd, actions/checkout@v1→50fbc622, haya14busa/action-bumpr@v1→faf6f474, haya14busa/action-update-semver@v1→7d2c5586, haya14busa/action-cond@v1→94f77f7a, actions/create-release@v1→0cb9c9b6, reviewdog/action-eslint@v1→556a3fda.

2. script-injection: Moved all 5 ${{ steps.*.outputs.value }} expressions in test.yml run: steps into env: blocks (TESTVAL, FALSEVAL, EVENT_VALUE) and referenced them as plain shell variables.

3. missing-permissions: Added top-level permissions blocks to all three files — release.yml gets contents:write (for tag/release creation), reviewdog.yml gets contents:read + checks:write + pull-requests:write (for reviewdog reporting), test.yml gets contents:read.

