# doc-hygiene - convention

## Semantic line breaks (long-form prose)
- Wrap at sentence/clause boundaries (~100 chars). Never write a paragraph as one physical line.
- A single newline inside a Markdown paragraph renders as a space - the output is identical, but
  the source stays readable, diffable line-by-line, and loadable by file tools.
- Never start a wrapped line with a block trigger: `-` `*` `+` `>` `|` `#` or `N.`
  (it would turn into a list/heading/quote). Never end a line with two trailing spaces (a hard `<br>`).
- Tables and code fences stay on one line.

## No em-dash in non-English UI copy
- Use a regular hyphen `-` in UI copy in the project's non-English language. Never an em-dash.
- English spec/prose may keep em-dashes.

## Leanness
- Always-loaded docs (the memory index, the status docs) stay lean; detail lives in topic/archive files.
