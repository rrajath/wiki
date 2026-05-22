Run the `audit` operation as defined in `.claude/SKILL.md`.

1. Read `CLAUDE.md` and `wiki/index.md` first.
2. Scan the `audit/` directory for open feedback files (those not in `audit/resolved/`).
3. For each feedback file:
   - Read the correction or note.
   - Locate the relevant wiki page(s).
   - Apply the correction.
   - Move the feedback file to `audit/resolved/`.
4. If there are contradictions between sources, state both perspectives, cite each source, and add the question to the "Open research questions" section of `CLAUDE.md`.
5. Append a session entry to `log/YYYYMMDD.md` summarising what was audited and resolved.

If no open feedback files are found, say so and stop.
