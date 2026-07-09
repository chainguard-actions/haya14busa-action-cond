<!-- markdownlint-disable -->

# Hardening Report: haya14busa--action-cond/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **haya14busa--action-cond/v1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references in the workflow files use mutable tags or branch names instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks. Failing references: release.yml: actions/checkout@v2, haya14busa/action-bumpr@v1, haya14busa/action-update-semver@v1, haya14busa/action-cond@v1; reviewdog.yml: actions/checkout@v1, reviewdog/action-eslint@v1; test.yml: actions/checkout@v2 (×2).

Locations:

- `.github/workflows/release.yml:13`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/reviewdog.yml:11`
- `.github/workflows/reviewdog.yml:13`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:17`

### missing-permissions (severity: medium)

None of the workflow files define a top-level permissions: key, and no job within them defines job-level permissions. This means workflows run with the default (potentially broad) token permissions. Affected files: release.yml, reviewdog.yml, test.yml.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/reviewdog.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Multiple run: blocks in test.yml directly interpolate ${{ steps.*.outputs.value }} expressions (sub-rule a). The steps.*.outputs.* context is workflow-controllable and flows through YAML template substitution before the shell sees it, enabling script injection. Offending lines include: `run: test "${{ steps.testval.outputs.value }}" = "true value"`, `run: test "${{ steps.falseval.outputs.value }}" = "false value"`, `run: echo "${{ steps.event.outputs.value }}"`, `run: test "${{ steps.event.outputs.value }}" = "value for pull request event"`, `run: test "${{ steps.event.outputs.value }}" = "value for non pull request event"`. These should be moved to env: variables and the env vars used in the shell script instead.

Locations:

- `.github/workflows/test.yml:24`
- `.github/workflows/test.yml:34`
- `.github/workflows/test.yml:46`
- `.github/workflows/test.yml:50`
- `.github/workflows/test.yml:54`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across .github/workflows/release.yml, reviewdog.yml, and test.yml:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHAs with tag comments:
   - actions/checkout@v2 → @ee0669bd1cc54295c223e0bb666b733df41de1c5 (release.yml, test.yml ×2)
   - actions/checkout@v1 → @50fbc622fc4ef5163becd7fab6573eac35f8462e (reviewdog.yml)
   - haya14busa/action-bumpr@v1 → @faf6f474bcb6174125cfc569f0b2e24cbf03d496
   - haya14busa/action-update-semver@v1 → @7d2c558640ea49e798d46539536190aff8c18715
   - haya14busa/action-cond@v1 → @94f77f7a80cd666cb3155084e428254fea4281fd
   - reviewdog/action-eslint@v1 → @556a3fdaf8b4201d4d74d406013386aa4f7dab96

2. missing-permissions: Added top-level permissions blocks:
   - release.yml: contents: write (needed for gh release create)
   - reviewdog.yml: contents: read
   - test.yml: contents: read

3. script-injection: Moved all 5 ${{ steps.*.outputs.value }} expressions from run: blocks into env: blocks (TESTVAL, FALSEVAL, EVENT_VALUE) and referenced them as plain shell variables.

