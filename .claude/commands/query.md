Run the `query` operation as defined in `.claude/SKILL.md`.

1. Read `CLAUDE.md` and `wiki/index.md` first.
2. Read the user's question (provided after this command).
3. Search the wiki pages in `wiki/concepts/`, `wiki/entities/`, and `wiki/summaries/` for relevant content.
4. If the answer is found, respond clearly and cite the specific wiki page(s).
5. If the answer is partially found, provide what's known and note the gaps.
6. If the answer is not in the wiki, say so — and offer to ingest new sources or flag it as an open research question.
7. Save the query and answer to `outputs/queries/YYYYMMDD-<slug>.md` for future reference.
