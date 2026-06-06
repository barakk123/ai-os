# Memory Index

> The always-loaded index - the ONLY memory file read in full every session (up to a token cap).
> One line per memory: a `[Title](file.md)` link + a one-sentence recall hook (WHEN is this useful?).
> Keep it LEAN - it has a size budget; bloat means the index loads only partially = silent memory loss.
> Detail NEVER lives here; it lives in the memory files, loaded on demand. Compact before you near the cap.

<!--
ONE LINE PER MEMORY. Format:
  - [Title](file.md) - one-sentence recall hook (phrase it as the trigger condition, not a bare label)

Native types (in each file's frontmatter `metadata.type`):
  user      - who the person is / standing preferences about them
  feedback  - how to WORK: a durable workflow rule, with the why
  project   - ongoing work state / constraints / decisions NOT derivable from code or git
  reference - a pointer to an external resource (doc / register / URL)

If an entry needs more than one hook sentence, the extra content belongs in the FILE, not here.
-->

<!-- Seeded by bootstrap (replace the examples below with real entries): -->

- [Shared Language Glossary](feedback_shared_language_glossary.md) - the project shorthand (external oracle,
  derivable, standing gate, ...); read once so the terms mean the same thing every session.
- [Pre-Commit Audit](feedback_pre_commit_audit.md) - before every commit/push, reconcile all prompts since the last
  commit against what was actually written to files; fix gaps first.
- [No Finding Dissolves](feedback_no_finding_dissolves.md) - every review finding gets a durable tracked record;
  "done" is a dispositioned ledger + zero un-triaged findings, never a test count.
- [Master Spec](project_master_spec.md) - the single source of truth is the master spec doc; update only it going forward.
- [Tracker Location](reference_tracker_location.md) - the deferrals + findings registry is docs/TRACKER.md; read it
  BEFORE proposing any new deferral.
