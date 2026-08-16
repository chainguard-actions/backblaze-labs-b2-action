<!-- markdownlint-disable -->

# Hardening Report: backblaze-labs--b2-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **backblaze-labs--b2-action/v1.1.0** was hardened automatically. 0 finding(s) were identified and resolved across 1 iteration(s).

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the 'Resolve immutable release tag' step of the validate job, added explicit newline sanitization before writing to $GITHUB_OUTPUT. Introduced `SAFE_REF=$(printf '%s' "$REQUESTED_REF" | tr -d '\n\r')` after all validation checks pass, then replaced all uses of `$REQUESTED_REF` in the $GITHUB_OUTPUT heredoc block with `$SAFE_REF`. The regex validation guard was already present but the finding required the explicit `tr -d '\n\r'` sanitization pipeline, which is now in place.

