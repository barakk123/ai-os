# platform-notes - convention

## Purpose
A living list of environment & tooling gotchas + their verified, non-destructive fixes.
Seeded with cross-project ones; the project appends its own as they surface.

## The seed set (keep what applies)
- **Corporate TLS-MITM**: keep verification - git `http.sslBackend=schannel`;
  pnpm/npm `NODE_OPTIONS=--use-system-ca` (Node 22+). Never disable verification.
- **Windows / PowerShell**: here-strings; no `&&` chaining; UTF-16 default encoding;
  a git commit-message here-string must contain NO double quotes (they split the arg - use `-F`).
- **CI**: pin tool/action versions (avoid `version: latest`).
- **Isolate edits from fragile shell**: a non-zero exit in a batched command cancels the whole
  batch and drops file edits - keep edits in their own step and verify each landed.

## Rule
A fix recorded here must KEEP safety (no disabling TLS verification, no `--force` shortcuts)
unless the owner explicitly rules otherwise. Record the fix WITH its rationale.
