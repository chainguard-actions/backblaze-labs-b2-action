<!-- markdownlint-disable -->

# Hardening Report: backblaze-labs--b2-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **backblaze-labs--b2-action/v1.0.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference external actions using mutable version tags (e.g., @v6, @v7, @v9) instead of pinned 40-character commit SHAs. This exposes the workflows to supply-chain attacks if a tag is moved to a malicious commit. Affected actions include: actions/checkout@v6, pnpm/action-setup@v6, actions/setup-node@v6, actions/upload-artifact@v7, lycheeverse/lychee-action@v2, actions/upload-pages-artifact@v5, actions/deploy-pages@v5, actions/github-script@v9, softprops/action-gh-release@v3.

Locations:

- `.github/workflows/ci.yml:20`
- `.github/workflows/daily-smoke.yml:34`
- `.github/workflows/docs-lint.yml:34`
- `.github/workflows/docs.yml:28`
- `.github/workflows/example-cache-artifacts.yml:30`
- `.github/workflows/example-cross-bucket-replicate.yml:24`
- `.github/workflows/example-deploy-site.yml:20`
- `.github/workflows/example-head.yml:26`
- `.github/workflows/example-hide-unhide.yml:24`
- `.github/workflows/example-inventory-and-cleanup.yml:24`
- `.github/workflows/example-promote-release.yml:24`
- `.github/workflows/example-purge.yml:30`
- `.github/workflows/example-scheduled-backup.yml:28`
- `.github/workflows/example-share-build-artifact.yml:22`
- `.github/workflows/example-sse-encryption.yml:26`
- `.github/workflows/example-verify-artifacts.yml:24`
- `.github/workflows/release.yml:43`

### unsafe-shell (severity: high)

The 'Download actionlint' step uses bash process substitution to pipe remote content directly to bash: `bash <(curl https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash)`. This fetches and executes an unverified remote script in a single step without downloading to a file first, verifying integrity, or pinning to a specific commit SHA.

Locations:

- `.github/workflows/ci.yml:97`

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. `run: ${{ steps.get_actionlint.outputs.executable }} -color` — the entire command is a GitHub Actions expression. If the step output were attacker-influenced, this would allow arbitrary command execution. Even for trusted outputs, any ${{ }} in a run: block is a script-injection risk because the value is substituted before the shell parses it.

Locations:

- `.github/workflows/ci.yml:100`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks directly interpolate ${{ steps.*.outputs.* }} expressions inside shell commands. These values are substituted by the Actions template engine before the shell parses them, allowing shell metacharacters in step outputs to break out of the intended command context. Affected patterns include: `test -n "${{ steps.link.outputs.presigned-url }}"`, `test "${{ steps.ls.outputs.files-listed }}" -ge 5`.

Locations:

- `.github/workflows/daily-smoke.yml:112`

### script-injection (severity: high)

Sub-rule (a): run: block directly interpolates ${{ steps.copy.outputs.file-id }} inside a shell test command: `test -n "${{ steps.copy.outputs.file-id }}"`.

Locations:

- `.github/workflows/example-cross-bucket-replicate.yml:52`

### script-injection (severity: high)

Sub-rule (a): run: block directly interpolates multiple ${{ steps.probe.outputs.* }} and ${{ steps.up.outputs.* }} expressions inside shell test and echo commands, including piping to jq: `echo '${{ steps.probe.outputs.summary-json }}' | jq -e ...`. A malicious value in a step output could inject shell commands.

Locations:

- `.github/workflows/example-head.yml:55`

### script-injection (severity: high)

Sub-rule (a): run: blocks directly interpolate ${{ steps.ls-hidden.outputs.files-listed }} and ${{ steps.ls-visible.outputs.files-listed }} inside shell test commands: `test "${{ steps.ls-hidden.outputs.files-listed }}" = "0"` and `test "${{ steps.ls-visible.outputs.files-listed }}" = "1"`.

Locations:

- `.github/workflows/example-hide-unhide.yml:57`
- `.github/workflows/example-hide-unhide.yml:79`

### script-injection (severity: high)

Sub-rule (a): run: block directly interpolates ${{ steps.inv.outputs.files-listed }} and ${{ steps.inv.outputs.summary-json }} inside shell echo and test commands: `echo "Files listed: ${{ steps.inv.outputs.files-listed }}"` and `test "${{ steps.inv.outputs.files-listed }}" = "5"`.

Locations:

- `.github/workflows/example-inventory-and-cleanup.yml:55`

### script-injection (severity: high)

Sub-rule (a): run: block directly interpolates ${{ steps.promote.outputs.file-id }} inside a shell test command: `test -n "${{ steps.promote.outputs.file-id }}" || { echo "no file-id"; exit 1; }`.

Locations:

- `.github/workflows/example-promote-release.yml:52`

### script-injection (severity: high)

Sub-rule (a): run: blocks directly interpolate ${{ steps.dry.outputs.files-deleted }}, ${{ steps.purge.outputs.files-deleted }}, and ${{ steps.ls.outputs.files-listed }} inside shell test commands.

Locations:

- `.github/workflows/example-purge.yml:68`
- `.github/workflows/example-purge.yml:82`
- `.github/workflows/example-purge.yml:97`

### script-injection (severity: high)

Sub-rule (a): run: blocks directly interpolate ${{ steps.save.outputs.file-id }} and ${{ steps.save.outputs.content-sha1 }} inside shell echo commands: `echo "Saved file-id: ${{ steps.save.outputs.file-id }}"`.

Locations:

- `.github/workflows/example-cache-artifacts.yml:60`

### script-injection (severity: high)

Sub-rule (a): run: block directly interpolates ${{ steps.drift.outcome }} inside a shell conditional: `if [ "${{ steps.drift.outcome }}" != "failure" ]`. Step outcome values are substituted before shell parsing.

Locations:

- `.github/workflows/example-verify-artifacts.yml:68`

### github-env-injection (severity: high)

The 'Upload with SSE-C' step writes the inherited env var SSE_C_KEY to $GITHUB_ENV without sanitization: `echo "SSE_C_KEY=$SSE_C_KEY" >> "$GITHUB_ENV"`. SSE_C_KEY is set from `${{ secrets.B2_SSE_C_KEY_B64 }}` (an env: mapping) or from `cat sse_c_key.b64`. Writing an env var to GITHUB_ENV without `printf '%s' "$SSE_C_KEY" | tr -d '\n\r'` allows a value containing newlines to inject additional environment variable definitions into subsequent steps.

Locations:

- `.github/workflows/example-sse-encryption.yml:67`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, unsafe-shell, script-injection, github-env-injection

**Notes:**

Fixed all 13 findings across 17 workflow files:

1. unpinned-uses: Pinned all 9 external actions to full 40-char commit SHAs with tag comments in all affected workflow files (ci.yml, daily-smoke.yml, docs-lint.yml, docs.yml, example-cache-artifacts.yml, example-cross-bucket-replicate.yml, example-deploy-site.yml, example-head.yml, example-hide-unhide.yml, example-inventory-and-cleanup.yml, example-promote-release.yml, example-purge.yml, example-scheduled-backup.yml, example-share-build-artifact.yml, example-sse-encryption.yml, example-verify-artifacts.yml, release.yml).

2. unsafe-shell (ci.yml): Replaced `bash <(curl ...)` with a two-step approach: download script to file with `curl -sSfL ... -o download-actionlint.bash`, then execute `bash download-actionlint.bash` separately.

3. script-injection (ci.yml, daily-smoke.yml, example-cross-bucket-replicate.yml, example-head.yml, example-hide-unhide.yml, example-inventory-and-cleanup.yml, example-promote-release.yml, example-purge.yml, example-cache-artifacts.yml, example-verify-artifacts.yml): Moved all ${{ steps.*.outputs.* }} and ${{ steps.*.outcome }} expressions out of run: shell commands into step-level env: blocks, referencing them as plain environment variables in the shell scripts.

4. github-env-injection (example-sse-encryption.yml): Added sanitization of SSE_C_KEY before writing to GITHUB_ENV using `safe=$(printf '%s' "$SSE_C_KEY" | tr -d '\n\r')` and writing `$safe` instead of the raw value.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed both findings in `.github/workflows/example-scheduled-backup.yml`:
1. **script-injection** (line 35): Moved `${{ github.run_id }}` out of the `run:` block into a step-level `env:` block as `RUN_ID: ${{ github.run_id }}`. The shell script now references `$RUN_ID` instead of the direct expression interpolation.
2. **github-env-injection** (line 36): Added sanitization before writing to `$GITHUB_OUTPUT`. The key value is now sanitized with `safe_key=$(printf '%s' "$key" | tr -d '\n\r')` and `safe_key` is written to the output instead of the raw `$key`, preventing newline injection attacks.

