<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1.0.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag/version refs instead of pinned full SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. release.yml unpinned refs: actions/checkout@v2 (line 13), haya14busa/action-bumpr@v1 (line 19), haya14busa/action-update-semver@v1 (line 24), haya14busa/action-cond@v1 (line 31), actions/create-release@v1 (line 38). reviewdog.yml unpinned refs: actions/checkout@v1 (line 12), reviewdog/action-eslint@v1 (line 16). test.yml unpinned refs: actions/checkout@v2 (line 12), actions/checkout@v2 (line 19).

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:24`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:38`
- `.github/workflows/reviewdog.yml:12`
- `.github/workflows/reviewdog.yml:16`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:19`

### script-injection (severity: high)

Multiple run: steps in test.yml directly interpolate ${{ steps.*.outputs.value }} expressions into shell commands (sub-rule a). The steps.* context is workflow-controllable and flows through YAML template substitution before the shell processes it, enabling script injection if the output value contains shell metacharacters. Offending lines: line 29: run: test "${{ steps.testval.outputs.value }}" = "true value"; line 36: run: test "${{ steps.falseval.outputs.value }}" = "false value"; line 44: run: echo "${{ steps.event.outputs.value }}"; line 48: run: test "${{ steps.event.outputs.value }}" = "value for pull request event"; line 51: run: test "${{ steps.event.outputs.value }}" = "value for non pull request event". Fix: move the step output into an env: variable and reference it as a quoted shell variable.

Locations:

- `.github/workflows/test.yml:29`
- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:44`
- `.github/workflows/test.yml:48`
- `.github/workflows/test.yml:51`

### missing-permissions (severity: medium)

None of the workflow files define a top-level permissions: key, and no individual job defines a permissions: key. Without explicit permissions, workflows run with the default (often write) token permissions, granting broader access than necessary. All three workflow files are affected: release.yml, reviewdog.yml, and test.yml.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across release.yml, reviewdog.yml, and test.yml:

1. unpinned-uses: Pinned all 9 action references to full commit SHAs using lookup_action_sha, preserving original tags as comments.

2. script-injection: Moved all 5 ${{ steps.*.outputs.value }} expressions in test.yml run: steps into env: blocks (TESTVAL, FALSEVAL, EVENT_VALUE), referencing them as plain shell variables.

3. missing-permissions: Added top-level permissions blocks — release.yml gets 'contents: write' (for creating releases/tags), reviewdog.yml gets 'contents: read, checks: write, pull-requests: write' (for ESLint reporting), and test.yml gets 'contents: read'.

