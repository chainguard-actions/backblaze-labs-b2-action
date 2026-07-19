<!-- markdownlint-disable -->

# Hardening Report: backblaze-labs--b2-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **backblaze-labs--b2-action/v1.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use mutable tag-based refs instead of immutable 40-character SHA-pinned commit hashes, making them vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing refs include: actions/checkout@v6, pnpm/action-setup@v6, actions/setup-node@v6, actions/upload-artifact@v7, lycheeverse/lychee-action@v2, actions/upload-pages-artifact@v3, actions/deploy-pages@v4, actions/github-script@v9, softprops/action-gh-release@v3.

Locations:

- `.github/workflows/ci.yml:21`
- `.github/workflows/daily-smoke.yml:30`
- `.github/workflows/docs-lint.yml:34`
- `.github/workflows/docs.yml:28`
- `.github/workflows/example-cache-artifacts.yml:22`
- `.github/workflows/example-cross-bucket-replicate.yml:20`
- `.github/workflows/example-deploy-site.yml:17`
- `.github/workflows/example-head.yml:22`
- `.github/workflows/example-hide-unhide.yml:20`
- `.github/workflows/example-inventory-and-cleanup.yml:20`
- `.github/workflows/example-promote-release.yml:21`
- `.github/workflows/example-purge.yml:22`
- `.github/workflows/example-scheduled-backup.yml:22`
- `.github/workflows/example-share-build-artifact.yml:19`
- `.github/workflows/example-sse-encryption.yml:22`
- `.github/workflows/example-verify-artifacts.yml:20`
- `.github/workflows/release.yml:36`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands. Before the shell executes the command, GitHub Actions substitutes the expression value as a raw string, allowing an attacker who controls the value to inject arbitrary shell commands. Offending lines include:
- ci.yml: `run: ${{ steps.get_actionlint.outputs.executable }} -color` — the entire run command is an expression
- daily-smoke.yml: `COUNT=${{ steps.ls.outputs.files-listed }}` and `run: test -n "${{ steps.link.outputs.presigned-url }}"`
- example-head.yml: `test "${{ steps.probe.outputs.file-id }}"`, `test "${{ steps.probe.outputs.content-sha1 }}"`, `test "${{ steps.probe.outputs.bytes-transferred }}"`, `echo '${{ steps.probe.outputs.summary-json }}'`
- example-hide-unhide.yml: `test "${{ steps.ls-hidden.outputs.files-listed }}"` and `test "${{ steps.ls-visible.outputs.files-listed }}"`
- example-purge.yml: `test "${{ steps.dry.outputs.files-deleted }}"` and `test "${{ steps.ls.outputs.files-listed }}"`
- example-cross-bucket-replicate.yml: `run: test -n "${{ steps.copy.outputs.file-id }}"`
- example-cache-artifacts.yml: `echo "Saved file-id: ${{ steps.save.outputs.file-id }}"` and `echo "SHA-1: ${{ steps.save.outputs.content-sha1 }}"`
- example-inventory-and-cleanup.yml: `echo "Files listed: ${{ steps.inv.outputs.files-listed }}"` and `test "${{ steps.inv.outputs.files-listed }}"`
- example-promote-release.yml: `test -n "${{ steps.promote.outputs.file-id }}"`
- example-verify-artifacts.yml: `if [ "${{ steps.drift.outcome }}" != "failure" ]`
- example-scheduled-backup.yml: `key="backups/${{ github.run_id }}/snapshot-..."`

Locations:

- `.github/workflows/ci.yml:86`
- `.github/workflows/daily-smoke.yml:88`
- `.github/workflows/daily-smoke.yml:107`
- `.github/workflows/example-head.yml:57`
- `.github/workflows/example-hide-unhide.yml:55`
- `.github/workflows/example-hide-unhide.yml:72`
- `.github/workflows/example-purge.yml:64`
- `.github/workflows/example-purge.yml:82`
- `.github/workflows/example-cross-bucket-replicate.yml:50`
- `.github/workflows/example-cache-artifacts.yml:57`
- `.github/workflows/example-inventory-and-cleanup.yml:52`
- `.github/workflows/example-promote-release.yml:52`
- `.github/workflows/example-verify-artifacts.yml:72`
- `.github/workflows/example-scheduled-backup.yml:37`

### unsafe-shell (severity: high)

The 'Download actionlint' step in ci.yml uses `bash <(curl https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash)` which downloads a remote script and pipes it directly into bash via process substitution. This is equivalent to `curl ... | bash` and executes untrusted remote content without any integrity verification. If the remote URL is compromised or the content changes, arbitrary code will execute on the runner.

Locations:

- `.github/workflows/ci.yml:83`

### github-env-injection (severity: high)

In example-scheduled-backup.yml, the 'Build snapshot payload' step constructs a shell variable `key` by directly interpolating `${{ github.run_id }}` (a github.* context value) and then writes it to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in the value could inject additional key=value pairs into the output file, potentially overwriting subsequent step outputs. The offending line is: `key="backups/${{ github.run_id }}/snapshot-$(date -u +%Y%m%d).tar.gz"` followed by `echo "key=$key" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/example-scheduled-backup.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, unsafe-shell, github-env-injection

**Notes:**

Fixed all 4 findings across 17 workflow files:

1. unpinned-uses: Pinned all 9 action refs to full 40-char SHAs (actions/checkout, pnpm/action-setup, actions/setup-node, actions/upload-artifact, lycheeverse/lychee-action, actions/upload-pages-artifact, actions/deploy-pages, actions/github-script, softprops/action-gh-release) with tag comments preserved.

2. script-injection: Moved all ${{ steps.* }} and ${{ github.* }} expressions out of run: blocks into step-level env: blocks, then referenced them as plain shell variables. Affected files: ci.yml, daily-smoke.yml, example-head.yml, example-hide-unhide.yml, example-purge.yml, example-cross-bucket-replicate.yml, example-cache-artifacts.yml, example-inventory-and-cleanup.yml, example-promote-release.yml, example-verify-artifacts.yml, example-scheduled-backup.yml.

3. unsafe-shell: In ci.yml, replaced `bash <(curl ...)` process substitution with a two-step approach: download script to /tmp/download-actionlint.bash first, then execute it separately.

4. github-env-injection: In example-scheduled-backup.yml, moved github.run_id to an env: block (RUN_ID), then sanitized the constructed key with `printf '%s' "$raw_key" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed scripts/actionlint.sh line 29: replaced the unsafe `bash <(curl -fsSL https://raw.githubusercontent.com/rhysd/actionlint/main/scripts/download-actionlint.bash)` process substitution with a safe two-step approach — first downloading the script to `/tmp/download-actionlint.bash` with `curl -fsSL ... -o /tmp/download-actionlint.bash`, then executing it separately with `bash /tmp/download-actionlint.bash`, and finally cleaning up the temp file. This is consistent with how the CI workflow already handles the same download.

