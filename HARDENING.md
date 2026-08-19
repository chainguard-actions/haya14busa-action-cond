<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if a tag is moved or an action is compromised.

release.yml: actions/checkout@v2, haya14busa/action-bumpr@v1, haya14busa/action-update-semver@v1, haya14busa/action-cond@v1, actions/create-release@v1
reviewdog.yml: actions/checkout@v1, reviewdog/action-eslint@v1
test.yml: actions/checkout@v2 (used twice)

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:35`
- `.github/workflows/reviewdog.yml:12`
- `.github/workflows/reviewdog.yml:19`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:18`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual job defines its own `permissions:` block. This means workflows run with GitHub's default (broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in test.yml directly interpolate `${{ steps.*.outputs.value }}` expressions inside shell commands (sub-rule a). The `steps.*` context is workflow-controllable and flows through YAML template substitution before the shell processes it, allowing an attacker to inject shell metacharacters. The action's output value is derived from user-supplied `if_true`/`if_false` inputs, making this exploitable via a crafted workflow call.

Offending lines:
- Line 28: `run: test "${{ steps.testval.outputs.value }}" = "true value"`
- Line 37: `run: test "${{ steps.falseval.outputs.value }}" = "false value"`
- Line 46: `run: echo "${{ steps.event.outputs.value }}"`
- Line 49: `run: test "${{ steps.event.outputs.value }}" = "value for pull request event"`
- Line 51: `run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"`

Fix: move the value into an env var and reference it as `"$ENV_VAR"` in the shell command.

Locations:

- `.github/workflows/test.yml:28`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:49`
- `.github/workflows/test.yml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/release.yml, .github/workflows/reviewdog.yml, and .github/workflows/test.yml:

1. unpinned-uses: Pinned all 9 action references to full 40-char SHAs (resolved via lookup_action_sha), preserving original tags as inline comments.

2. missing-permissions: Added top-level permissions blocks — release.yml gets `contents: write` (for creating releases/tags), reviewdog.yml gets `contents: read` + `checks: write` + `pull-requests: write` (for reviewdog PR review comments), test.yml gets `contents: read`.

3. script-injection: Fixed all 5 offending run steps in test.yml by moving each `${{ steps.*.outputs.value }}` expression into the step's `env:` block (as TESTVAL, FALSEVAL, or EVENT_VALUE) and referencing it as `"$VAR"` in the shell command.

