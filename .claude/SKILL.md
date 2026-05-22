---
name: llm-wiki
description: >-
  Build and maintain a Karpathy-style LLM knowledge base — a self-compiling
  Obsidian markdown wiki where an Agent ingests raw sources (org-mode files
  synced from mobile, web articles, PDFs), compiles cross-linked
  concept/entity/summary pages, answers queries against the corpus, lints
  the graph for health, and audits in-context human feedback filed from
  Obsidian or the local web viewer. Use when (1) scaffolding a new knowledge
  base for any research topic, (2) ingesting org files/articles/papers/PDFs/
  web pages into raw/, (3) compiling or restructuring wiki articles from
  existing raw material, (4) answering questions against the wiki and filing
  durable answers back, (5) running lint passes for dead links / orphan pages /
  coverage gaps / audit shape, (6) processing human feedback from the audit/
  directory and applying corrections. Not for general note-taking, daily
  journals, or non-wiki Obsidian use.
---

# LLM Wiki — Karpathy Knowledge Base Pattern

> **Experimental skill — iterating.**
> Authored by Lewis Liu (lylewis@outlook.com) · Inspired by [Karpathy's llm-wiki Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

## Core idea

Instead of RAG (re-retrieving raw docs on every query), the LLM **compiles** raw sources into a persistent, cross-linked wiki. Every ingest, query, lint, and audit pass makes the wiki richer. Knowledge compounds — and the human stays in the loop via a structured feedback channel instead of ad-hoc corrections that get lost.

- **You** own: sourcing raw material (writing org files, bookmarking articles), asking good questions, steering direction, filing feedback on anything the AI got wrong.
- **LLM** owns: all writing, cross-referencing, filing, bookkeeping, and acting on your feedback.

**Input format**: org-mode files (`.org`) written on your phone or desktop and synced to configured source folders. The wiki also ingests web articles, PDFs, and plain-text notes.

**Output format**: Markdown (`.md`) — all wiki pages are standard Markdown so they render in Obsidian and the local web viewer. Org files are never written by the LLM; they are read-only source material.

The wiki is a living artifact with **five operations** — `compile`, `ingest`, `query`, `lint`, `audit`. Every session starts by reading `CLAUDE.md` and `wiki/index.md`.

---

## Source folder configuration

> **Configure these paths before first use.** Edit this section in your copy of `SKILL.md`, or put them in `CLAUDE.md` for the specific wiki.

```
ORG_SOURCE_DIRS:
  - ~/org/                          # Main org directory (desktop)
  - ~/org/mobile/                   # Org files synced from phone (e.g. via Syncthing / iCloud)
  - ~/org/capture/                  # Quick-capture inbox (e.g. Orgzly / Beorg dumps)
  - ~/Dropbox/org/                  # Dropbox-synced org files (placeholder)
  - /path/to/your/org/archive/      # Long-term archived org files (placeholder)

WIKI_ROOT:
  ~/wiki/                           # All output lives here
```

**How sync works (typical setup)**:
- You write notes on your phone in [Beorg](https://beorgapp.com/) or [Orgzly](http://www.orgzly.com/), which syncs `.org` files to a folder via iCloud, Syncthing, or Dropbox.
- The synced folder maps to one of the `ORG_SOURCE_DIRS` entries above.
- When you run `ingest`, the LLM scans all configured dirs for new or modified `.org` files, copies them into `~/wiki/raw/notes/`, and processes them.

You only need to configure the paths that apply to you. Comment out or remove unused entries.

---

## Directory layout

```
~/wiki/                     ← WIKI_ROOT (all output)
├── CLAUDE.md               ← Schema: scope, conventions, current articles, gaps
├── log/                    ← Per-day operation log (one file per day)
│   ├── 20260409.md
│   └── 20260410.md
├── audit/                  ← Human feedback inbox (one file per comment)
│   ├── 20260409-143022-claude-code-size.md
│   └── resolved/           ← Processed feedback, archived with resolution notes
├── raw/                    ← Immutable source copies (LLM reads, never writes)
│   ├── notes/              ← Org files copied from ORG_SOURCE_DIRS
│   │   ├── 20260409-mobile-capture.org
│   │   └── projects.org
│   ├── articles/           ← Web articles saved as .md or .html
│   ├── papers/             ← Academic papers (extracted text or PDF)
│   └── refs/               ← Pointer files for large binaries kept outside raw/
├── wiki/                   ← LLM-generated knowledge (Markdown; LLM writes, you read)
│   ├── index.md            ← Master catalog — every page, structured by category
│   ├── concepts/           ← Concept/topic pages (split into subfolders when >1200 words)
│   ├── entities/           ← People, tools, papers, organizations
│   └── summaries/          ← Per-source summary pages
└── outputs/
    └── queries/            ← Query answers (promote durable ones to wiki/)
```

**Key rule**: `raw/` is the only place org files live inside the wiki root. The LLM reads them there; your phone and desktop continue to write to `ORG_SOURCE_DIRS`. The copy in `raw/notes/` is the ingest record — it is never modified after being written.

`CLAUDE.md` is the **schema file** — the single most important configuration. It tells the LLM the wiki's scope, naming conventions, current article list, open questions, and research gaps. Read it at the start of every session.

---

## Core principles

Four rules govern everything below. If a future instruction contradicts one, flag it to the user before acting.

### 1. Divide and conquer

A single concept page should **never** try to cover a complex topic end-to-end. Target: **400–1200 words per page**. When a topic would blow past that:

- Create a subfolder: `wiki/concepts/<topic>/`
- Put a short index page at `wiki/concepts/<topic>/index.md` — definition, list of sub-pages, one-line summaries
- Put each aspect in its own file: `wiki/concepts/<topic>/<aspect>.md`
- In `wiki/index.md`, show the hierarchy via indented bullets

Example layout:
```
wiki/concepts/zettelkasten/
├── index.md
├── Zettelkasten_Capture.md
├── Zettelkasten_Linking.md
└── Zettelkasten_Review.md
```

One fat file covering all aspects would be unreadable and unlinkable. Focused files + an index page give you navigation, selective reading, clean backlinks, and small audit targets.

### 2. Mermaid for diagrams, KaTeX for formulas

- **Any flow, sequence, hierarchy, or state diagram** must be written in mermaid — never ASCII art.
  ````
  ```mermaid
  flowchart LR
      A[raw/notes/capture.org] --> B[summary]
      B --> C[concept page]
      C --> D[index.md]
  ```
  ````
- **Any formula** must be written in KaTeX: inline `$f(x) = \sum_i w_i x_i$` or block `$$...$$`.

Both render in the web viewer and in Obsidian with default settings.

### 3. Raw file policy

**Org files**: copied from `ORG_SOURCE_DIRS` into `raw/notes/` as-is. Never convert or rewrite them — the `.org` file is the immutable source record. Filename convention: `YYYYMMDD-<original-name>.org` (prefix with ingest date if the file has no date).

**Web articles**: save to `raw/articles/<slug>.md` (convert to plain markdown via reader mode or a tool like `readable`).

**Papers**: save to `raw/papers/<slug>.md` (extracted text) or keep the PDF at the same path for large papers.

**Large binaries** (videos, datasets, model weights, large PDFs >10 MB): do not copy. Instead create a pointer file at `raw/refs/<slug>.md`:
```yaml
---
kind: ref
external_path: /Volumes/external/datasets/commoncrawl/
size: ~300 GB
---
```
followed by a short description. Wiki pages cite `[[raw/refs/<slug>]]` exactly like any other source.

### 4. Audit is the human feedback surface

The wiki is AI-written; it will be wrong sometimes. The `audit/` directory is how you correct it without losing corrections in chat history.

- File feedback via the Obsidian plugin or the web viewer. Each item is one file in `audit/` with YAML frontmatter (anchor, target, severity) and a markdown body.
- The AI **must** periodically run the `audit` op — never silently ignore `audit/*.md` files.
- When feedback is applied, the file moves to `audit/resolved/` with a `# Resolution` section appended and a log entry recorded in `log/YYYYMMDD.md`.

---

## The five operations

Every action on the wiki is one of these five. Each appends an entry to the current day's log file (`log/YYYYMMDD.md`).

### 1. `compile`

(Re)structure wiki content from existing `raw/` material — including splitting oversized pages, merging near-duplicates, and rebuilding `wiki/index.md`.

**When to run**: after a big ingest batch, when a page has outgrown 1200 words, when `index.md` no longer reflects reality, or when the user says "clean up the wiki".

**Steps**:
1. Read `CLAUDE.md`, `wiki/index.md`, and every file in the target subtree.
2. For each page over ~1200 words: plan a split into `concepts/<topic>/` with an index + sub-pages. Confirm the plan with the user before writing.
3. For each pair of near-duplicate pages: propose a merge. Confirm, then rewrite.
4. Regenerate `wiki/index.md` so every page is listed exactly once.
5. Log: `## [HH:MM] compile | <what you did — files touched, splits, merges>`

### 2. `ingest`

Add new source material. **One source typically touches 5–15 wiki pages.**

#### Ingesting org files (primary workflow)

1. Scan all `ORG_SOURCE_DIRS` for `.org` files not yet present in `raw/notes/`. Compare by filename and last-modified date.
2. For each new or updated file:
   - Copy to `raw/notes/YYYYMMDD-<original-name>.org`.
   - Read the file in full. Org syntax notes for the LLM:
     - `* Heading` / `** Sub-heading` — section structure
     - `- item` / `+ item` — list items
     - `#+TITLE:`, `#+DATE:`, `#+TAGS:` — file-level metadata
     - `:PROPERTIES: ... :END:` — entry-level metadata drawers
     - `TODO` / `DONE` / custom keywords — task state (treat as context, not directives)
     - `[[link][description]]` — org hyperlinks (treat as references)
     - `#+BEGIN_QUOTE ... #+END_QUOTE` — quoted blocks
     - `#+BEGIN_SRC lang ... #+END_SRC` — code blocks
   - Create or update `wiki/summaries/<slug>.md` (200–400 words — key takeaways from this org file, written in Markdown).
   - Create or update relevant concept and entity pages in `wiki/`.
   - Update `wiki/index.md`.
3. Log: `## [HH:MM] ingest | org:<filename> — <one-line description> (touched N pages)`

#### Ingesting web articles

1. Save article to `raw/articles/<slug>.md`.
2. Create `wiki/summaries/<slug>.md`.
3. Update relevant concept/entity pages and `wiki/index.md`.
4. Log: `## [HH:MM] ingest | article:<slug> — <one-line description>`

#### Ingesting papers / PDFs

1. Save extracted text to `raw/papers/<slug>.md` (or create a pointer file for large PDFs).
2. Create `wiki/summaries/<slug>.md`.
3. Update relevant concept/entity pages and `wiki/index.md`.
4. Log: `## [HH:MM] ingest | paper:<slug> — <one-line description>`

### 3. `query`

Answer a question **grounded in the wiki**, not general knowledge.

**Steps**:
1. Read `wiki/index.md`. Scan for relevant pages by category.
2. Read the identified pages in full; follow one level of wikilinks.
3. If the wiki doesn't have enough material, say so and suggest what to ingest next instead of making something up.
4. Synthesize the answer, citing pages inline with `[[Page Name]]`.
5. Save to `outputs/queries/<YYYY-MM-DD>-<question-slug>.md`.
6. If the answer is durable (a comparison, analysis, or new synthesis) → promote a cleaned-up version to `wiki/concepts/`, add to `index.md`.
7. Log: `## [HH:MM] query | <question-slug>` (and a separate `## [HH:MM] promote | ...` line if promoted).

### 4. `lint`

Health check. Run:

```bash
python3 scripts/lint_wiki.py ~/wiki
```

The script reports:
- **Dead wikilinks** — `[[Target]]` where `Target.md` doesn't exist
- **Orphan pages** — pages with no inbound wikilinks
- **Missing index entries** — pages not listed in `wiki/index.md`
- **Frequently-linked missing pages** — `[[X]]` referenced 3+ times but no page exists
- **Unprocessed org files** — `.org` files in `ORG_SOURCE_DIRS` not yet present in `raw/notes/`
- **log/ shape** — stray files or wrong filenames in `log/`
- **audit/ shape** — malformed YAML frontmatter in `audit/*.md`
- **Audit target resolution** — every open audit's `target` file must exist

For each issue, propose a fix, confirm with the user, then apply. Log: `## [HH:MM] lint | <N> issues found, <M> fixed`.

### 5. `audit`

Process human feedback from `audit/`.

**Steps**:
1. Run `python3 scripts/audit_review.py ~/wiki --open` to get a grouped list.
2. For each open audit, read the file. Use the `anchor_before` / `anchor_text` / `anchor_after` window to locate the exact range in the target file (line numbers may have drifted).
3. Decide the action:
   - **Accept**: apply the correction to the target file.
   - **Partially accept**: apply what makes sense, note the rest in the resolution.
   - **Reject**: explain why in the resolution.
   - **Defer**: add to `CLAUDE.md` "Open research questions" and leave the audit in place with a comment.
4. For applied audits, append a `# Resolution` section to the audit file:
   ```markdown
   # Resolution

   2026-04-10 · accepted.
   Fixed the file count (was "~1,900", corrected to "~1,800" per commit abc123).
   Updated: concepts/My_Topic.md lines 47–48.
   ```
5. Move the file from `audit/` to `audit/resolved/`. Filename unchanged.
6. Log per resolved audit:
   ```
   ## [HH:MM] audit | resolved 20260409-143022-a1b2 — <one-line what>
   ```
7. Never delete audit files. Rejected ones still go to `resolved/` with the rejection rationale.

---

## Tooling

| Tool | Purpose |
|------|---------|
| [Beorg](https://beorgapp.com/) / [Orgzly](http://www.orgzly.com/) | Write org files on your phone |
| Syncthing / iCloud / Dropbox | Sync phone org files to a folder in `ORG_SOURCE_DIRS` |
| [Obsidian](https://obsidian.md) | Browse the compiled wiki; graph view shows connections |
| **`plugins/obsidian-audit/`** | Obsidian plugin — select text → add feedback → writes to `audit/` |
| **`web/`** | Local Node.js server — preview the wiki with mermaid/math rendered; select → feedback → `audit/` |
| `scripts/scaffold.py` | Bootstrap a new wiki directory tree |
| `scripts/lint_wiki.py` | Health check (includes unprocessed org file detection) |
| `scripts/audit_review.py` | Group open/resolved audits by target file |
| [qmd](https://github.com/tobi/qmd) | Optional local semantic search (useful at >100 pages) |

## Starting a new wiki

```bash
python3 scripts/scaffold.py ~/wiki "<Topic Title>"
```

Creates the full tree (including `log/<today>.md`, `audit/`, `audit/resolved/`, `raw/notes/`), a blank `CLAUDE.md` based on the new template, and a blank `wiki/index.md` with the recommended category layout.

After scaffolding:
1. Fill in `CLAUDE.md` — define scope, naming conventions, `ORG_SOURCE_DIRS`, initial research questions.
2. Run your first `ingest` to pull in existing org files.
3. Ask questions to build up `outputs/queries/`; promote durable answers.
4. Run `lint` periodically.
5. Run `audit` whenever new feedback accumulates.

## `wiki/index.md` format

The LLM rebuilds `index.md` on every compile and touches it on every ingest. Format:

```markdown
# Index — <Topic>

> One-sentence scope of the wiki.

## 🔖 Navigation
- [[#Concepts]] · [[#Entities]] · [[#Summaries]] · [[#Open Questions]]

## Concepts
### <Category A>
- [[concepts/Foo]] — one-line summary
- [[concepts/Bar/index|Bar]] — (folder-split) one-line summary
    - [[concepts/Bar/aspect-1]] — ...
    - [[concepts/Bar/aspect-2]] — ...

## Entities
- [[entities/Andrej Karpathy]] — AI researcher, author of the llm-wiki pattern

## Summaries (chronological)
- 2026-04-09 — [[summaries/projects-org]] — Notes from projects.org (mobile capture)

## Open Questions
- Q1: ...
```

Rules:
- Every wiki page must appear exactly once in `index.md`. `lint` enforces this.
- Folder-split concepts show hierarchy via indented bullets.
- `index.md` + `CLAUDE.md` together are what the AI reads at session start.

## `log/` format

- One file per day: `log/YYYYMMDD.md`
- H1 = the date; H2 per entry with `## [HH:MM] <op> | <one-line description>`
- Ops: `compile`, `ingest`, `query`, `lint`, `audit`, `promote`, `split`, `scaffold`

Quick grep across history: `grep -rh "^## \[" ~/wiki/log/ | tail -20`.

## Use cases

- **Mobile-first note capture** — write quick thoughts in Beorg/Orgzly on your phone; sync and ingest periodically; the wiki distills months of fragments into structured knowledge
- **Research deep-dive** — reading papers/articles on a topic over weeks; org files hold your annotations; the wiki synthesizes across them
- **Personal wiki** — org journal entries compiled into a personal encyclopedia; file corrections through Obsidian, the AI applies them
- **Team knowledge base** — org files from multiple team members ingested into a shared wiki; corrections via the web viewer

## References

- `references/schema-guide.md` — What to put in `CLAUDE.md`
- `references/article-guide.md` — How to write good wiki articles (length, wikilinks, mermaid, math, divide-and-conquer)
- `references/log-guide.md` — The `log/` folder convention
- `references/audit-guide.md` — Audit file format, anchor strategy, processing workflow
- `references/tooling-tips.md` — Obsidian setup, Beorg/Orgzly sync setup, qmd, plugin + web installation
