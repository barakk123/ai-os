# Platform Notes - <project>

> Environment & tooling gotchas + their verified, non-destructive fixes.
> Keep what applies; append the project's own. Fixes must keep safety unless the owner rules otherwise.

## Environment
- **OS / shell:** <e.g. Windows 10 / PowerShell 5.1; bash also available>

## Known gotchas + fixes
- **Corporate TLS-MITM proxy** breaks cert verification. Keep verification:
  git `http.sslBackend=schannel`; pnpm/npm `NODE_OPTIONS=--use-system-ca` (Node 22+).
  Do NOT disable verification.
- **PowerShell**: here-strings for multi-line; no `&&` chaining; UTF-16 default encoding
  (`-Encoding utf8` when other tools read the file); no double quotes in a git commit here-string.
- **CI**: pin tool/action versions (no `version: latest`).
- **Isolate edits from fragile shell**: a non-zero exit cancels a batched command and drops edits.

## Project-specific (append below)
- <...>
