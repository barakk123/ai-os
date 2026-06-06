# tracker - convention

## The ledger (docs/TRACKER.md)
One row per entry, with a stable id:

| Field | Values |
|---|---|
| `id` | `T-XXXX` (sequential, never reused) |
| `category` | `deferral` · `finding` · `user-action` |
| `severity` | `high` · `medium` · `low` |
| `source` | where it came from (a run, a review, an owner ruling) |
| `status` | `open` · `in-progress` · `resolved` |
| `summary` | one line; detail / links as needed |

## Rules
- **no-finding-dissolves**: every deferral AND finding becomes a row. Nothing lives only in chat.
- **scan-before-deferring**: before proposing a new deferral, scan the open rows for the area
  you are about to touch.
- **"done" is never a count**: an item is done when its rows are dispositioned + its gates met,
  not when a number is hit.
- **Resolved section**: move resolved rows to a separate Resolved section so the live ledger
  stays short (roll to a dated archive if it grows).
