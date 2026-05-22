Run the `lint` operation as defined in `.claude/SKILL.md`.

1. Read `CLAUDE.md` and `wiki/index.md` first.
2. Scan all pages in `wiki/` and check for:
   - **Dead links**: internal links that point to pages that don't exist.
   - **Orphan pages**: pages not linked from any other page or from `wiki/index.md`.
   - **Missing frontmatter**: pages lacking required fields (`title`, `type`, `created`, `updated`, `sources`, `tags`).
   - **Oversized pages**: pages exceeding ~1200 words that haven't been split.
   - **Coverage gaps**: concepts or entities mentioned in multiple pages but lacking their own dedicated page.
3. Report all issues grouped by type.
4. For straightforward fixes (missing frontmatter fields, dead links to pages that exist under a different name), fix them directly.
5. Append a session entry to `log/YYYYMMDD.md` listing issues found and actions taken.
