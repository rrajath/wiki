Run the `compile` operation as defined in `.claude/SKILL.md`.

1. Read `CLAUDE.md` and `wiki/index.md` first.
2. Review all pages in `wiki/concepts/`, `wiki/entities/`, and `wiki/summaries/`.
3. For each page:
   - Check that frontmatter is complete (`title`, `type`, `created`, `updated`, `sources`, `tags`).
   - Ensure cross-links between related pages are present and accurate.
   - Confirm page length is within 400–1200 words; split into a subfolder if over.
   - Verify all diagrams use Mermaid and all formulas use KaTeX.
4. Update `wiki/index.md` with an accurate list of all current articles.
5. Update the "Current articles" section of `CLAUDE.md`.
6. Append a session entry to `log/YYYYMMDD.md` summarising what was compiled and changed.
