# platform-notes

> A starter set of environment & tooling gotchas to pre-empt, plus a place for the project to
> record its own (OS, shell, proxy, CI).

**Status:** since v0.1
**Dependencies:** `standalone`.

<!-- ai-os:manifest
tier: core            # core | heavy
deps: { hard: [], soft: [] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- Known cross-project gotchas (corporate TLS-MITM, Windows/PowerShell, CI version pinning,
  the isolate-edits-from-fragile-shell lesson).
- A `CLAUDE.md` "Platform Notes" section the project extends with its own.

## Install

Paste into the project's `CLAUDE.md` (keep what applies, add the project's own):

> ### Platform Notes
> - **Corporate TLS-MITM proxy** breaks OpenSSL cert verification. Fixes that KEEP verification:
>   git `http.sslBackend=schannel`; pnpm/npm `NODE_OPTIONS=--use-system-ca` (Node 22+, Windows
>   trust store). Do NOT disable verification (strict-ssl false / NODE_TLS_REJECT_UNAUTHORIZED=0).
> - **Windows + PowerShell**: here-strings for multi-line; no `&&` chaining; UTF-16 default
>   encoding (pass `-Encoding utf8` when other tools must read the file). A git commit-message
>   here-string must contain NO double quotes - they split the arg; use `git commit -F <file>`.
> - **CI**: pin tool/action versions (a `version: latest` setup step rate-limits and flakes).
> - **Isolate edits from fragile shell**: never bundle a file Edit/Write with fragile shell
>   (`grep -c`, arithmetic) in one batch - a non-zero exit cancels the batch and silently drops the
>   edits. Edit in isolation; verify each landed; check git status before committing.

## Generates in target

- A "Platform Notes" section in `CLAUDE.md` (seeded; the project extends it).

## Files it scaffolds

- `templates/platform-notes.template.md` - the seed list above, to copy + tailor.

## Why

The same environment traps cost real time on every project on a given machine (a TLS proxy that
blocks installs, a shell quirk, a CI flake, a tooling batch that silently drops edits). Capturing
them once means a new project on the same setup inherits the fixes instead of re-discovering them.
