# Karpathy LLM Wiki vs Clarity Knowledge Systems

#decisions #architecture #wiki #knowledge-management

Comparison of Andrej Karpathy's "LLM Wiki" pattern (published April 2026) against Clarity's three knowledge systems: expertise.yaml, .claude/memory/, and wiki/. Research conducted 2026-04-06.

Sources: [Karpathy's gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), [Extended Brain Substack analysis](https://extendedbrain.substack.com/p/the-wiki-that-writes-itself), [VentureBeat coverage](https://venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an).

---

## Part 1: Karpathy's Architecture (Exact Details)

### Three-Layer Structure

```
project/
  raw/                  # Layer 1: Immutable source documents
    assets/             # Downloaded images (Obsidian attachment folder)
    *.md, *.pdf, etc.   # Articles, papers, data files -- LLM reads, never writes
  wiki/                 # Layer 2: LLM-generated and LLM-maintained
    index.md            # Content catalog -- every page with one-line summary, by category
    log.md              # Append-only chronological record of ingests, queries, lint passes
    [entity pages]      # e.g. people, organizations, technologies
    [concept pages]     # e.g. theories, patterns, frameworks
    [summary pages]     # Source summaries, comparisons, analyses
  CLAUDE.md             # Layer 3: Schema -- tells LLM how wiki is structured, conventions, workflows
```

### Processing Pipeline

**Ingest (raw -> wiki):**
1. User drops source document into `raw/`
2. User tells LLM to process it
3. LLM reads source, discusses key takeaways with user
4. LLM writes a summary page in wiki/
5. LLM updates `index.md` with new page listing
6. LLM updates relevant entity and concept pages across wiki (10-15 pages per source)
7. LLM appends entry to `log.md`
8. User reviews in Obsidian in real-time

**Query (wiki -> answer -> wiki):**
1. User asks a question
2. LLM reads `index.md` first to locate relevant pages
3. LLM drills into those pages, follows links
4. LLM synthesizes answer with citations
5. Good answers get filed back into wiki as new pages (this is key -- questions compound)

**Lint (wiki -> wiki):**
1. Periodic health check, user-initiated
2. LLM scans for contradictions between pages
3. Flags stale claims superseded by newer sources
4. Identifies orphan pages with no inbound links
5. Suggests missing pages for important concepts
6. Identifies missing cross-references
7. Recommends web searches to fill data gaps

### How Relationships/Links Are Built

- Standard Obsidian `wiki links` syntax
- LLM identifies semantic connections during compilation
- Cross-references created when related concepts appear across multiple sources
- Synthesis articles bridge conceptual gaps
- No vector similarity -- relies on LLM reasoning to identify meaningful relationships
- Obsidian graph view shows connection topology visually

### How the Index Works

- `index.md` is a plain-text catalog, not a database
- Every wiki page listed with link + one-line summary
- Organized by category (entities, concepts, sources)
- Updated on every ingest operation
- LLM reads index first during queries to locate relevant pages
- Works at moderate scale (~100 sources, hundreds of pages, ~400k words)
- Fits in modern context windows -- no embedding infrastructure needed

### The CLAUDE.md Schema

Karpathy's gist is intentionally abstract about the schema. It serves as:
- Configuration file telling the LLM how the wiki is structured
- Documents conventions, page formats, workflows
- Defines what happens during ingest, query, and lint operations
- Co-evolved between user and LLM over time
- Named CLAUDE.md for Claude Code, AGENTS.md for Codex, etc.
- This is the "key configuration file" that makes the LLM a disciplined wiki maintainer

### What Makes It Different From RAG

| Aspect | RAG | Karpathy Wiki |
|--------|-----|---------------|
| Knowledge state | Re-derived on every query | Compiled once, kept current |
| Accumulation | Nothing compounds | Everything compounds |
| Cross-references | Rebuilt each time via retrieval | Already exist in wiki pages |
| Contradictions | Discovered per-query (maybe) | Flagged during lint passes |
| Synthesis | Generated on the fly | Pre-built, incrementally refined |
| Infrastructure | Vector DB, embeddings, retrieval pipeline | Plain markdown files + index |
| Transparency | Embeddings are black box | Every claim traceable to a .md file |
| Scale ceiling | Scales well | Works to ~100 sources / ~400k words, then needs search tooling |

### Tooling (Optional, Not Core)

- **Obsidian**: IDE for browsing wiki, graph view, Dataview plugin for frontmatter queries
- **Obsidian Web Clipper**: Browser extension to convert web articles to markdown
- **Marp**: Markdown slide decks from wiki content
- **qmd**: Local markdown search with hybrid BM25/vector search + LLM re-ranking (CLI + MCP)
- **Git**: Version history for free since it is just markdown files

---

## Part 2: Clarity's Three Knowledge Systems

### System 1: expertise.yaml (Per-Client Structured Knowledge)

```yaml
# 324 lines for alba-netchb
meta: { client, phase, last_updated, se_lead }
solution: { description, architecture, scale_tested }
platform_state: { tenant, object_types, agents, flows, service }
netchb_sandbox: { url, endpoints, operations }
filing_results: { totals, latest, samples }
implementation_patterns: { event_chain, config_routing, ... }
known_issues: [ ... ]
unvalidated_observations: [ ... ]
```

**How it works:**
- Every `/se:*` command reads it for context and appends observations
- `/se:self-improve` validates observations against live state, promotes confirmed facts
- Structured YAML with defined sections -- not freeform
- Capped at 1000 lines per client
- Must always be valid YAML (machine-parseable)
- Per-client scope only -- no cross-client knowledge

**Strengths:** Machine-readable, structured, fits in any context window, directly actionable by commands.
**Weaknesses:** Rigid schema, hard to capture nuanced relationships, no cross-references, per-client silo.

### System 2: .claude/memory/ (Cross-Session Memory)

```
.claude/projects/{project}/memory/
  MEMORY.md              # Index -- one-line descriptions with links
  feedback_*.md          # User preferences and process rules (14 files)
  project_*.md           # Project state snapshots (12 files)
  strategy_*.md          # Strategic context (1 file)
  user_profile.md        # Who the user is
```

Each file has YAML frontmatter:
```yaml
---
name: Stop overcomplicating - save progress
description: CRITICAL process failure - not saving lessons learned
type: feedback
---
```

**How it works:**
- Claude automatically reads MEMORY.md at session start
- Files loaded into context as system reminders
- Created/updated by Claude during conversations when important patterns emerge
- Cross-project (each project has its own memory folder)
- No explicit ingest pipeline -- Claude decides what is worth remembering

**Strengths:** Automatic, zero-effort for user, carries behavioral rules and preferences, cross-session persistence.
**Weaknesses:** Claude-managed (user does not control index), no wiki-links between entries, no lint/health-check process, entries can go stale (8-day warning shown), flat structure.

### System 3: wiki/ (Obsidian-Compatible Knowledge Base)

```
wiki/
  index.md               # Category listing with Dataview query
  log.md                 # Processing log (date, source, pages created)
  platform/              # 7 pages -- flow-structure, schema-rules, triggers, etc.
  clients/               # 1 page -- alba-netchb overview
  patterns/              # 6 pages -- event-chain, cron-polling, etc.
  decisions/             # 3 pages -- architectural decisions with rationale
  people/                # 4 pages -- team members, roles
```

**How it works:**
- wiki links between pages (e.g., flow-structure links to triggers, error-handling, etc.)
- Tags on every page (#platform #flows #architecture)
- Created 2026-04-06 -- brand new, seeded from expertise.yaml + session notes
- Obsidian-compatible graph view
- No automated ingest pipeline yet
- No query-back-to-wiki loop yet
- No lint/health-check process yet

**Strengths:** Human-readable, relationship-rich, browsable in Obsidian, flexible structure.
**Weaknesses:** Just created, no automated workflows, manually seeded, no raw/ pipeline, no log discipline.

---

## Part 3: What Karpathy Does That We Don't

### 1. Automated Ingest Pipeline (raw/ -> wiki/)
Karpathy has a defined workflow: drop a file in `raw/`, tell the LLM, it processes and updates 10-15 pages. We have a `raw/` directory but it is empty and has no processing pipeline. Our wiki was manually seeded from expertise.yaml, not from raw source documents.

### 2. Query-to-Wiki Filing
When Karpathy asks a good question and gets a synthesis answer, that answer gets filed back into the wiki as a new page. Our answers disappear into chat history. This is the compounding mechanism -- exploration becomes permanent knowledge.

### 3. Lint/Health-Check Operations
Karpathy periodically runs the LLM over the entire wiki to find contradictions, orphans, stale claims, and missing pages. We have nothing like this. Our `/se:self-improve` validates observations in expertise.yaml against live state, but it does not audit the wiki for structural health.

### 4. Disciplined Log
Karpathy's `log.md` is append-only with parseable prefixes (`## [2026-04-02] ingest | Article Title`). Ours has a single entry in a table format -- not a living log, just a creation record.

### 5. The Schema File as Contract
Karpathy's CLAUDE.md (or equivalent) is a co-evolved contract between user and LLM defining exactly how ingest, query, and lint work. Our CLAUDE.md is project documentation -- it describes commands and rules but does not define wiki-specific operational workflows.

### 6. Index as Navigation Aid
Karpathy's index.md lists every page with a one-line summary, organized by category. The LLM reads it first on every query. Our index.md lists categories with folder links and has a Dataview query but no per-page summaries -- not useful as an LLM navigation aid.

### 7. Source Traceability
Every Karpathy wiki page traces back to specific raw sources. Our wiki pages cite "expertise.yaml" or "session notes" but do not link to original raw documents.

### 8. Explicit Separation of Human vs LLM Roles
Karpathy is explicit: human curates sources and asks questions, LLM does everything else. We blur this line -- some wiki pages were manually guided, expertise.yaml is partially manual, memory is fully automatic.

---

## Part 4: What We Do That Karpathy Doesn't

### 1. Machine-Readable Structured Knowledge (expertise.yaml)
Karpathy's wiki is markdown for humans and LLMs. Our expertise.yaml is structured YAML that commands can parse programmatically. `/se:scaffold` reads it to auto-fill templates. `/se:check` validates against it. This is not just knowledge -- it is operational data.

### 2. Self-Improve Loop Against Live State
Our `/se:self-improve` validates observations against actual live systems (Jira ticket status, tenant record counts, Slack context). Karpathy's lint checks internal consistency but does not validate against external truth.

### 3. Command-Driven Knowledge Pipeline
Every `/se:*` command reads from and writes to expertise.yaml automatically. The knowledge system is integrated into the workflow, not a separate activity. Karpathy's ingest requires explicit "process this" commands.

### 4. Behavioral Memory (.claude/memory/)
Our memory system carries user preferences, process rules, and critical guardrails (like "never create schemas from scratch" or "stop overcomplicating"). These are injected into every session automatically. Karpathy has no equivalent -- his schema file defines wiki behavior, not user preferences.

### 5. Multi-Tenant/Multi-Client Architecture
We have per-client knowledge isolation (each client gets its own expertise.yaml + wiki pages) plus cross-project memory. Karpathy's pattern is single-domain.

### 6. Deployment Chain Integration
Our knowledge system is tied to actual platform operations -- deploy cycles, API gotchas, service dependencies. It is not just a research wiki; it drives real engineering work.

### 7. Phase-Gated Process
Our Phase 0 discovery, `/se:check` compliance, and structured engagement workflow do not exist in Karpathy's pattern. He is building a research knowledge base; we are building an engineering operations system.

---

## Part 5: How to Bridge the Gap

### Priority 1: Add Ingest Pipeline (High Impact, Low Effort)

Create a processing workflow for `raw/` -> `wiki/`:

1. Add a `/se:wiki-ingest` command (or hook it into existing commands)
2. When a file lands in `raw/`, the command:
   - Reads the source
   - Creates/updates relevant wiki pages
   - Updates `index.md` with per-page summaries
   - Appends to `log.md` with parseable format
   - Cross-links to existing pages
3. Sources to ingest: meeting notes (from `/se:meeting`), Jira exports, Slack threads, architecture reviews, API discovery sessions

**Implementation:** Add to CLAUDE.md as a wiki operations section. Define the ingest workflow explicitly.

### Priority 2: Query-to-Wiki Filing (High Impact, Medium Effort)

When an LLM synthesis is valuable during a conversation:

1. File it as a new wiki page with source attribution
2. Update index.md
3. Add cross-links to related pages
4. Append to log.md

This could be a `/se:wiki-file` command or just a convention documented in CLAUDE.md.

### Priority 3: Wiki Lint/Health-Check (Medium Impact, Low Effort)

Add a `/se:wiki-lint` command that:

1. Scans all wiki pages for orphans (no inbound links)
2. Identifies stale pages (not updated in 2+ weeks)
3. Finds contradictions between wiki pages and expertise.yaml
4. Suggests missing pages for concepts referenced but not documented
5. Checks that index.md is complete and accurate

### Priority 4: Upgrade index.md (Medium Impact, Low Effort)

Change index.md from category-folder listing to Karpathy-style per-page catalog:

```markdown
## Platform
- [[platform/flow-structure]] -- node types, lint rules, deploy cycle, entryPoint critical
- [[platform/triggers]] -- contextual-start conditions, event delivery
- [[platform/schema-rules]] -- object type creation via management proxy
...
```

This makes index.md useful as an LLM navigation aid, not just a human directory.

### Priority 5: Enrich CLAUDE.md with Wiki Operations (Medium Impact, Low Effort)

Add a "Wiki Operations" section to CLAUDE.md defining:
- Ingest workflow (step by step)
- Query workflow (read index first, then drill into pages)
- Lint workflow (periodic health check)
- Filing convention (when to create a new wiki page from a conversation)
- Log format (`## [YYYY-MM-DD] type | description`)

---

## Part 6: Pros/Cons Comparison

### expertise.yaml

| Pros | Cons |
|------|------|
| Machine-readable, commands parse it | Rigid schema, hard to evolve |
| Directly actionable (scaffold, check, brief) | No cross-references or relationships |
| Self-improve loop validates against live state | Per-client silo, no cross-client patterns |
| Compact (fits in any context window) | Not human-browsable (YAML walls) |
| Always valid YAML (enforceable) | 1000-line cap forces compression |

**Best for:** Operational data that commands need to read. Tenant state, filing results, agent versions, known issues.

### .claude/memory/

| Pros | Cons |
|------|------|
| Automatic (zero user effort) | User does not control what gets remembered |
| Carries behavioral rules across sessions | No links between entries |
| Injected into every conversation | Can go stale (no refresh mechanism) |
| Lightweight (frontmatter + short prose) | Flat structure, no categories |
| Cross-project scope | No health-check or audit process |

**Best for:** User preferences, critical guardrails, process rules, session-level context. Things the LLM needs to remember about *how to work*, not *what it knows*.

### wiki/ (Obsidian-Compatible)

| Pros | Cons |
|------|------|
| Human-readable and browsable | No automated pipeline (yet) |
| Rich relationships via wiki links | Manually seeded, not systematically maintained |
| Obsidian graph view shows topology | No lint process (yet) |
| Flexible structure, easy to evolve | No query-to-wiki filing loop (yet) |
| Tags + Dataview for dynamic queries | Index is not LLM-navigable (yet) |
| Git-friendly (just markdown) | No raw/ ingest pipeline (yet) |

**Best for:** Synthesized knowledge, architectural patterns, decision records, people/team context, cross-cutting concepts. The kind of knowledge that benefits from relationships and narrative.

### Karpathy Wiki (Reference)

| Pros | Cons |
|------|------|
| Full automated pipeline (ingest, query, lint) | No structured/machine-readable layer |
| Query answers compound into wiki | Single-domain only |
| Lint keeps wiki healthy over time | No external validation (no live-state checks) |
| Plain-text index avoids RAG complexity | Scale ceiling at ~400k words without search tools |
| Source traceability to raw/ | No behavioral memory for LLM |
| Clear human/LLM role separation | No command-driven integration |

**Best for:** Research, topic deep-dives, personal knowledge accumulation. Domains where the goal is understanding, not operations.

---

## Part 7: Specific Recommendations for Clarity

### Keep All Three Systems (They Serve Different Purposes)

Do not collapse into a single system. Each has a distinct role:

- **expertise.yaml** = operational database (what the tenant looks like, what works, what is broken)
- **memory/** = behavioral rules (how to work with this user, critical guardrails)
- **wiki/** = synthesized knowledge (why things are the way they are, how concepts connect)

Karpathy has one system because he has one use case (research). We have three because we have three use cases (operations, behavior, knowledge). The right move is to connect them, not merge them.

### Connect the Three Systems

1. **expertise.yaml -> wiki/**: When expertise.yaml is updated with significant findings, create/update corresponding wiki pages. The wiki is the human-readable narrative; expertise.yaml is the machine-readable data.

2. **memory/ -> wiki/decisions/**: When a memory entry captures an architectural decision or critical learning, it should also exist as a wiki decision page with full rationale and links.

3. **wiki/ -> expertise.yaml**: When wiki lint finds that a wiki page contradicts expertise.yaml, flag it for resolution.

### Adopt Karpathy's Core Innovation: The Compounding Loop

The single most important thing Karpathy does is make questions compound. Every good answer becomes a wiki page. Every wiki page links to others. Every ingest updates many pages. The system gets smarter by being used.

To adopt this:
1. Add query-to-wiki filing as a convention
2. Add raw/ ingest pipeline
3. Add lint as a periodic operation
4. Upgrade index.md to be LLM-navigable

### Do Not Adopt: Karpathy's Lack of Structure

Karpathy's wiki is freeform markdown. For SE work, we need structure. Keep expertise.yaml as the structured layer. Keep the wiki as the narrative layer. Do not try to make the wiki replace expertise.yaml or vice versa.

### Timeline

| Action | Effort | Impact | When |
|--------|--------|--------|------|
| Upgrade index.md to per-page catalog | 30 min | High | Now |
| Add wiki operations section to CLAUDE.md | 1 hour | High | Now |
| Build `/se:wiki-ingest` from raw/ | 2 hours | High | This week |
| Add query-to-wiki filing convention | 30 min | High | This week |
| Build `/se:wiki-lint` | 1 hour | Medium | Next week |
| Connect expertise.yaml updates to wiki updates | 2 hours | Medium | Next week |
| Add raw/ sources (meeting notes, reviews, API docs) | Ongoing | High | Start now |

---

## Part 8: The Extended Brain Critique (Worth Noting)

The Substack analysis raises a valid concern from the Zettelkasten tradition: the wiki synthesizes sources but does not embody the user's own reasoning. The friction of struggling with difficult ideas is where understanding forms. Karpathy's system removes that friction.

For our use case, this critique is less relevant. We are not building a personal thinking tool -- we are building an SE operations system. The goal is not deep personal understanding of customs filing theory. The goal is that any SE can join an engagement and be productive on day one. The wiki should capture *what the team has learned*, not force each new SE to re-derive it.

That said, the critique does apply to architectural decisions. Decision pages in `wiki/decisions/` should capture the *reasoning*, not just the conclusion. The struggle of "why did we choose this?" is important institutional knowledge. This is why our decision pages already include rationale sections.

---

## Summary

Karpathy's LLM Wiki pattern and Clarity's knowledge systems solve different problems with overlapping techniques. The key gaps to close are:

1. **Ingest pipeline** (raw/ -> wiki/) -- we have the folders but no automation
2. **Compounding loop** (query answers filed back to wiki) -- our answers vanish into chat
3. **Lint process** (periodic health check) -- we validate expertise.yaml but not the wiki
4. **LLM-navigable index** (per-page summaries) -- ours is a folder listing

The key advantages to preserve are:

1. **Structured operational data** (expertise.yaml) -- Karpathy does not have this
2. **Behavioral memory** (.claude/memory/) -- Karpathy does not have this
3. **Live-state validation** (/se:self-improve) -- Karpathy does not have this
4. **Command-driven integration** (/se:* pipeline) -- Karpathy does not have this

The end state: three connected systems where expertise.yaml is the operational database, wiki/ is the synthesized knowledge base with Karpathy-style compounding, and memory/ is the behavioral layer. Each fed by different sources, each serving different consumers, all cross-referenced.

## Related

- [[platform/flow-structure]] -- example of a well-structured wiki page
- [[clients/alba-netchb]] -- example of expertise.yaml -> wiki page conversion
- [[patterns/event-chain]] -- example of cross-linked pattern documentation
