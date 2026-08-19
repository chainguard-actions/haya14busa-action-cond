<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ }} expressions referencing steps.*.outputs.* are interpolated directly inside run: shell command strings in test.yml. These values flow through YAML template substitution before the shell sees them, allowing an attacker to inject shell metacharacters. Offending lines:
- Line 28: run: test "${{ steps.testval.outputs.value }}" = "true value"
- Line 37: run: test "${{ steps.falseval.outputs.value }}" = "false value"
- Line 50: run: echo "${{ steps.event.outputs.value }}"
- Line 53: run: test "${{ steps.event.outputs.value }}" = "value for pull request event"
- Line 56: run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"
Fix: move the values into env: variables and reference them as quoted shell variables (e.g. "$STEP_OUTPUT").

Locations:

- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:50`
- `.github/workflows/test.yml:53`
- `.github/workflows/test.yml:56`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branch names rather than full 40-character commit SHAs. A compromised or malicious tag update could silently alter the code executed in CI. Failing references:
- release.yml: actions/checkout@v2 (line 13), haya14busa/action-bumpr@v1 (line 19), haya14busa/action-update-semver@v1 (line 23), haya14busa/action-cond@v1 (line 29), actions/create-release@v1 (line 35)
- reviewdog.yml: actions/checkout@v1 (line 12), reviewdog/action-eslint@v1 (line 19)
- test.yml: actions/checkout@v2 (line 12, line 19)
Fix: pin every uses: reference to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:35`
- `.github/workflows/reviewdog.yml:12`
- `.github/workflows/reviewdog.yml:19`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:19`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level permissions: key, and no job within any of them defines job-level permissions either. Without explicit permissions, workflows inherit the repository's default token permissions (often write-all), granting unnecessarily broad access. Affected files: release.yml, reviewdog.yml, test.yml. Fix: add a top-level permissions: block with the minimum required scopes (e.g. contents: read) to each workflow, and grant write scopes only to the specific jobs that need them.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across all three workflow files: (1) script-injection in test.yml — moved all 5 ${{ steps.*.outputs.value }} expressions from run: shell strings into env: blocks as STEP_VALUE, referencing them as "$STEP_VALUE" in the shell; (2) unpinned-uses — pinned all 9 mutable tag references to full 40-char SHAs with original tag in comment (actions/checkout@v2→0717577d, @v1→50fbc622, haya14busa/action-bumpr@v1→faf6f474, action-update-semver@v1→7d2c5586, action-cond@v1→94f77f7a, actions/create-release@v1→0cb9c9b6, reviewdog/action-eslint@v1→556a3fda); (3) missing-permissions — added top-level permissions blocks: contents:read for test.yml and reviewdog.yml, contents:write for release.yml (which creates releases/tags).

