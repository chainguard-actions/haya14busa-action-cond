<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **haya14busa--action-cond/v1.2.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag/version refs instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tags are moved or overwritten.

Failing references:
- release.yml: `actions/checkout@v2`, `haya14busa/action-bumpr@v1`, `haya14busa/action-update-semver@v1`, `haya14busa/action-cond@v1`
- reviewdog.yml: `actions/checkout@v1`, `reviewdog/action-eslint@v1`
- test.yml: `actions/checkout@v2` (appears twice)

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/reviewdog.yml:12`
- `.github/workflows/reviewdog.yml:19`
- `.github/workflows/test.yml:11`
- `.github/workflows/test.yml:20`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job within them defines job-level permissions either. Without explicit permissions, workflows inherit the default repository permissions (often `write-all`), granting unnecessarily broad access to the GITHUB_TOKEN.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in test.yml directly interpolate `${{ steps.*.outputs.value }}` expressions inside shell command strings (sub-rule a). The `steps.*.outputs.*` context is workflow-controllable and is substituted into the shell command before execution, allowing an attacker to inject arbitrary shell commands if the action's output can be influenced.

Offending lines:
- `run: test "${{ steps.testval.outputs.value }}" = "true value"` (line 30)
- `run: test "${{ steps.falseval.outputs.value }}" = "false value"` (line 40)
- `run: echo "${{ steps.event.outputs.value }}"` (line 51)
- `run: test "${{ steps.event.outputs.value }}" = "value for pull request event"` (line 54)
- `run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"` (line 57)

Fix: assign the output to an env var and reference it as a quoted shell variable, e.g.:
```yaml
env:
  STEP_VALUE: ${{ steps.testval.outputs.value }}
run: test "$STEP_VALUE" = "true value"
```

Locations:

- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:40`
- `.github/workflows/test.yml:51`
- `.github/workflows/test.yml:54`
- `.github/workflows/test.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/release.yml, reviewdog.yml, and test.yml:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHAs (actions/checkout@v2→0717577, @v1→50fbc62, haya14busa/action-bumpr@v1→faf6f47, haya14busa/action-update-semver@v1→7d2c558, haya14busa/action-cond@v1→94f77f7, reviewdog/action-eslint@v1→556a3fd). Original tags preserved as inline comments.

2. missing-permissions: Added top-level permissions blocks — release.yml gets 'contents: write' (for gh release create and tag pushing), reviewdog.yml gets 'contents: read, pull-requests: write, checks: write' (for reviewdog PR review and check annotations), test.yml gets 'contents: read'.

3. script-injection: All 5 offending run: steps in test.yml that interpolated ${{ steps.*.outputs.value }} directly into shell strings now use an env: block (STEP_VALUE: ${{ steps.*.outputs.value }}) and reference the value as "$STEP_VALUE" in the shell command.

