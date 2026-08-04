<!-- markdownlint-disable -->

# Hardening Report: backblaze-labs--b2-action/v1.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **backblaze-labs--b2-action/v1.2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Resolve immutable release tag' step of the 'validate' job, the env var REQUESTED_REF (set from `${{ github.event_name == 'workflow_dispatch' && inputs.tag || github.ref_name }}`) is written directly to $GITHUB_OUTPUT as `echo "release_tag=$REQUESTED_REF" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Both `inputs.tag` (a workflow_dispatch input) and `github.ref_name` are untrusted sources. A malicious tag name or dispatch input containing newline characters could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `.github/workflows/release.yml:100`

### github-env-injection (severity: high)

In the 'Derive major-version floating tag' step of the 'publish' job, the env var MAJOR is derived from RELEASE_TAG (which originates from `needs.validate.outputs.release_tag`, itself sourced from `inputs.tag` or `github.ref_name`) via `sed`. The result is written to $GITHUB_OUTPUT as `echo "major=$MAJOR" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although sed limits the output to `v` followed by digits, the check rules require explicit sanitization for any value transitively derived from untrusted inputs.

Locations:

- `.github/workflows/release.yml:347`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/.github/workflows/release.yml:
1. 'Resolve immutable release tag' step (line ~100): Added `SAFE_REF=$(printf '%s' "$REQUESTED_REF" | tr -d '\n\r')` and `SAFE_SHA=$(printf '%s' "$RELEASE_SHA" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, replacing the raw variable references with the sanitized versions.
2. 'Derive major-version floating tag' step (line ~347): Added `SAFE_MAJOR=$(printf '%s' "$MAJOR" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, replacing the raw `$MAJOR` reference with `$SAFE_MAJOR`.

