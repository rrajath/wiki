Run the `ingest` operation as defined in `.claude/SKILL.md`.

1. Read `CLAUDE.md` and `wiki/index.md` first.
2. Scan all org source directories listed in `CLAUDE.md` for `.org` files not yet present in `raw/notes/`.
3. For each new file:
   - Copy it into `raw/notes/`.
   - Summarize its content and extract key concepts and entities.
   - Update or create relevant wiki pages in `wiki/concepts/`, `wiki/entities/`, or `wiki/summaries/`.
4. Update `wiki/index.md` to reflect any new or changed pages.
5. Append a session entry to `log/YYYYMMDD.md` listing what was ingested.

If no new org files are found, say so and stop.
If there are many files, offer to ingest in batches of 5 to stay within context window limits.
