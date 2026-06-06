# doc-hygiene

> How long-form Markdown is written: semantic line breaks, no em-dash in non-English UI copy,
> and lean docs - so the source stays readable, diffable, and fully tool-loadable.

**Status:** since v0.1
**Dependencies:** `standalone`.

<!-- ai-os:manifest
version: 0.1.0        # bumps (semver) when this module's content changes; absence is fine (older installs have none)
tier: core            # core | heavy
deps: { hard: [], soft: [] }
tiers: [single]       # [single]  OR  [lite, full]  OR named e.g. [authored-spec, external, hybrid]
-->

---

## What it gives you

- The semantic-line-break rule (wrap at clause boundaries, never one giant physical line).
- The no-em-dash-in-non-English-UI rule.
- A leanness reminder (detail lives in topic/archive files, not in always-loaded docs).

## Install

Paste into the project's `CLAUDE.md`:

> ### Doc Hygiene
> Write long-form Markdown (status, specs, tracker, memory) with **semantic line breaks**: wrap at
> sentence/clause boundaries (~100 chars), never as one giant physical line. A single newline inside
> a paragraph renders as a space (identical output) but keeps the source readable, diffable, and
> tool-loadable. Never start a wrapped line with a block trigger (`-` `*` `+` `>` `|` `#` `N.`) and
> never end a line with two trailing spaces. Tables and code fences stay on one line. In non-English
> UI copy use a regular hyphen, never an em-dash (English spec prose may keep em-dashes).

## Generates in target

- A "Doc Hygiene" section in `CLAUDE.md`.

## Files it scaffolds

- None (behavioral).

## Why

Status/spec/memory files bloated into 54-60KB single physical lines that file tools could not load
and diffs could not show. Semantic line breaks keep the rendered output identical while making the
source maintainable; the em-dash rule retires a recurring UI-copy correction.
