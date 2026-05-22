# Personal Knowledge Base

> Schema document — read at the start of every session together with `wiki/index.md`.
> Update after every major compile, ingest batch, or structural change.

## Scope

What this wiki covers:
- Personal notes, reading highlights, and ideas captured in org-roam
- Concepts, entities, and summaries distilled from those notes
- Cross-linked knowledge graph built from daily capture

What this wiki deliberately excludes:
- Raw org files (those stay in the org source dirs below)
- Ephemeral todos or calendar/agenda items

## Org source directories

- ~/Library/CloudStorage/Dropbox/org-roam (only use the files in this directory. Don't do a recursive search. There is a folder called journals in the org-roam folder. I dont' want that folder ingested)
- ~/Library/CloudStorage/Dropbox/org-mode/thoughts.org
- ~/Library/CloudStorage/Dropbox/org-mode/resources.org
- ~/Library/CloudStorage/Dropbox/org-mode/blog.org


## Operations

This wiki follows the llm-wiki skill's five operations: `compile`, `ingest`, `query`, `lint`, `audit`.
Every operation appends an entry to `log/YYYYMMDD.md`.
Full operation instructions: `.claude/SKILL.md`

## Naming conventions

- **Concept pages** (`wiki/concepts/`): Title Case noun phrases.
- **Folder-split concepts** (`wiki/concepts/<topic>/`): used when a topic exceeds ~1200 words. Contains `index.md` + one file per aspect.
- **Entity pages** (`wiki/entities/`): Proper names.
- **Summary pages** (`wiki/summaries/`): kebab-case source slug.

All pages require YAML frontmatter: `title`, `type`, `created`, `updated`, `sources`, `tags`.

### Page size
- Target 400–1200 words per page.
- If a topic exceeds ~1200 words, split into a subfolder with `index.md` + one file per aspect.

### Diagrams and formulas
- All diagrams are **mermaid**. No ASCII art.
- All formulas are **KaTeX** (inline `$...$` or block `$$...$$`).

### Raw file policy
- Small text sources → copy into `raw/<subfolder>/`.
- Large binaries → create a pointer file at `raw/refs/<slug>.md` with `kind: ref` and `external_path` fields. Do not copy the binary.

## Current articles

Last updated: 2026-05-12 (first full ingest, 107 files).

### Concepts

**Career & Mastery**: Career Capital · Career Capital Market · Craftsman Mindset · Passion Mindset · Deliberate Practice · Make Little Bets · Skill Stacking · Goal Setting · Vision

**Planning & Decision Making**: Think Slow Act Fast · Commitment Fallacy · Reference Class Forecasting · Prioritization as a Knapsack Problem · Priorities are Time-Bound · You Always Make the Best Decision · Decisions vs Outcomes · 12 Week Year

**Systems Thinking**: Systems Thinking · Reinforcing Loop · Balancing Loop

**Productivity & Workflow**: Workflow · Productivity Improvement · App Summon · Easy Plug-and-Play · Muscle Memory

**Awareness & Cognition**: Awareness · Information Overload · Decision Fatigue · Mental Junk Food · Our Defaults · Instant Gratification vs Deep Satisfaction

**Stoicism & Mindset**: Stoicism · Dichotomy of Control · Accept Reality as It Is · Attaching Identity to Outcomes · Labels Anchor Us Down · Arbitrary Future State of Supposed Happiness

**Mental Models**: Map is not the Territory

**Learning**: Contemplative Reading

**Tools**: NixOS · Jujutsu

**IVF (folder-split)**: IVF/index · IVF/Menstrual Cycle · IVF/Frozen Embryo Transfer Protocols · IVF/IVF Pregnancy Complications

**Pregnancy**: Stages of Labor

**Distributed Systems (folder-split)**: Distributed Systems/index · CAP Theorem · DynamoDB · Cassandra · Redis · PostgreSQL · Database Indexing · Distributed Primitives · Load Balancer · Flink · System Design Numbers

### Entities
*(none)*

### Summaries

- so-good-they-cant-ignore-you
- 12-week-year
- how-big-things-get-done
- thoughts-org
- resources-org
- blog-org

## Open research questions

- What does a deliberate practice routine look like for knowledge work (writing, programming)?
- How do you identify which career capital market you're in?
- What are the specific placental complications in IVF and why does age accelerate risk?
- What are the precise stages of labor and the optimal epidural timing window?
- How does Jujutsu's operation log differ practically from Git reflog?

## Research gaps

Sources to ingest:
- [ ] Expand IVF pregnancy complications with clinical research
- [ ] Expand Stages of Labor with clinical detail
- [ ] Expand Jujutsu with tutorial notes after hands-on use
- [ ] Expand Bloom Filter internals (bit array, multiple hash functions)
- [ ] Expand Phi Accrual Failure Detector internals

## Audit backlog

*(none — run `python3 scripts/audit_review.py <wiki-root> --open` to refresh)*

## Notes for the LLM

- Language: <en | zh | bilingual>
- Tone: <neutral, academic, conversational, ...>
- Depth: <survey-level | deep technical>
- Handling contradictions: state both, cite each, add to Open Research Questions.
