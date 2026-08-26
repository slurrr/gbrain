# GBrain Skills Index

A human-oriented index of the skills available in this checkout. Use it to
choose what to load into an agent; use the individual `SKILL.md` as the actual
contract and `skills/RESOLVER.md` as the routing map.

**Snapshot:** 2026-08-26

**Available routed skills:** 69

**Canonical sources:** `skills/*/SKILL.md` frontmatter, then the skill body

## How to use this index

- Read the complete `SKILL.md` before invoking a skill.
- `mutating: true` means the workflow can write or change something; it is not
  a security boundary by itself.
- `writes_pages` / `writes_to` describe declared brain-page write surfaces.
- The resolver's trigger phrases are authoritative for routing. This index is
  for orientation and curation, not dispatch.
- Some skills are conventions or operational runbooks in prose and assume
  tools that a particular harness may not expose.

## Alex-specific recommendation

Alex is a creative, user-facing thinking partner with gbrain as durable memory.
The current Pi adapter exposes only these model-visible gbrain tools:

```text
recall, remember, entity, forget, get_page, put_page
```

The adapter calls `context_pack`, `delta`, and `volunteer_context` internally;
those should remain hidden from Alex. Most bundled skills were written for a
full CLI/MCP agent and declare classic tools such as `search`, `query`,
`list_pages`, `add_link`, `add_timeline_entry`, `sync_brain`, or `exec`. A skill
can be conceptually useful while still being **not directly executable** in the
current Alex surface.

### Best candidates for Alex

| Skill | Recommendation | Why |
|---|---|---|
| `brain-ops` | Foundation, adapted | Defines brain-first lookup, source attribution, read-enrich-write, and link discipline. Alex already implements much of this natively, but the skill declares more tools than Alex currently exposes. |
| `correction-pipeline` | Strong candidate | Gives a durable response to “that fact is wrong”: identify whether the problem is a fact, page, identity, or model error, then correct the source. Pairs naturally with `gbrain_forget`. |
| `idea-lineage` | Strong candidate, later | Excellent fit for tracing how one idea changed, including reversals, abandoned branches, and the current version. Needs a few additional read operations or an adapter translation. |
| `strategic-reading` | Strong candidate, on demand | Turns an article, book, transcript, or case study into an applied playbook for a specific problem. Fits Alex's creative-partner role, but should be invoked only when the user supplies a source and asks for durable analysis. |
| `book-mirror` | Strong candidate, on demand | Personalizes a book through brain context without turning the agent into a life consultant. Useful for deliberate creative reflection, not ordinary chat. |
| `brain-ingest-gate` | Useful guard | Prevents raw or duplicate material from entering the brain and routes new content through the right ingestion path. Useful if Alex begins accepting larger source inputs. |
| `brain-link-discipline` | Useful convention | Ensures reported brain pages have working links and that link provenance is handled correctly. Helpful whenever Alex delivers a saved artifact. |
| `data-loss-gate` | Useful safety convention | Requires an explicit confirmation before destructive brain or filesystem actions. Especially relevant now that Alex has `edit` and `write`. |
| `ask-user` | Useful convention | Gives Alex a disciplined choice gate instead of guessing when the user must choose among paths. |
| `resolve-before-asking` | Conditional | Good for uncertain people, roles, and relationships: exhaust brain evidence before asking the user. Use only when identity resolution becomes part of Alex's work. |
| `query` | Conceptually useful, adapter mismatch | Provides cited brain answers and graph-aware lookup, but it expects `search`, `query`, `list_pages`, timeline, and graph tools that Alex does not currently expose. |
| `capture` | Conceptually useful, adapter mismatch | A good one-entry capture workflow, but Alex currently has the narrower `remember` and `put_page` tools instead. |
| `concept-synthesis` | Background only | Valuable for turning many raw ideas into a tiered intellectual map, but it is a batch/dream-style maintenance workflow, not a per-turn Alex capability. |
| `context-audit` | Operator-only | Useful for auditing Alex's prompt and skill budget. It is report-only and should be run deliberately, not loaded as a normal conversational skill. |

### Keep out of Alex's default surface

- `signal-detector`: always-on capture is too broad for the current policy, which
  intentionally excludes ordinary brainstorm fragments and transcript capture.
- `conversation-archive` and `meeting-ingestion`: conflict with the current
  no-transcript/no-meeting-memory decision.
- `soul-audit`: Alex's identity is workspace-owned and intentionally not managed
  by gbrain bootstrap rendering.
- `briefing`, `daily-task-manager`, `daily-task-prep`, and `cron-scheduler`:
  personal-assistant operations outside Alex's role.
- `setup`, `cold-start`, `migrate`, `company-brainify`, and schema migration
  skills: operator/setup workflows, not conversational memory.
- Dream and maintenance skills should run in the separate native gbrain worker,
  not through Alex's Pi skill or tool surface.

## Complete catalog

The fit labels below mean:

- **Core:** plausible Alex capability after adapting the declared tools.
- **On demand:** useful for an explicit user request, not always loaded.
- **Background:** belongs in gbrain maintenance, dream, or a scheduled worker.
- **Operator:** setup, diagnosis, upgrades, or repo maintenance.
- **Meta:** creates, tests, routes, or improves skills.
- **Out of scope:** conflicts with Alex's current role or memory policy.

### Brain operations and memory

| Skill | What it does | Alex fit |
|---|---|---|
| `brain-ops` | Defines the brain-first read → enrich → write loop, attribution, ambient enrichment, and backlinks. | **Core / adapted** |
| `query` | Answers brain questions through layered search, page reads, graph context, citations, and explicit gaps. | **On demand / adapter mismatch** |
| `capture` | Provides one human-facing `gbrain capture` entrypoint for saving a thought or source. | **On demand / adapter mismatch** |
| `ingest` | Routes generic content ingestion to specialized ingestion skills. | **On demand / adapter mismatch** |
| `enrich` | Builds or updates person and company pages with compiled truth, timelines, and links. | **On demand / source-dependent** |
| `repo-architecture` | Decides where a new brain page belongs based on its primary subject. | **Useful reference** |
| `brain-taxonomist` | Reads the active schema pack and recommends the correct filing path before a page write. | **Useful later / operator** |
| `brain-ingest-gate` | Deduplicates and quality-gates content before it enters the brain; delegates enrichment. | **Useful later** |
| `correction-pipeline` | Traces a factual correction back to its source and prevents the same error recurring. | **Core candidate** |
| `brain-link-discipline` | Makes working page links part of the user-facing deliverable and protects link provenance. | **Core convention** |
| `maintain` | Audits brain health, citations, backlinks, stale information, orphans, and dream-cycle state. | **Background / operator** |
| `frontmatter-guard` | Validates and repairs YAML frontmatter, slugs, delimiters, and null-byte problems. | **Background / operator** |
| `citation-fixer` | Audits and repairs citation formatting and broken source references. | **Background** |
| `data-loss-gate` | Requires explicit confirmation before bulk deletion, purge, source removal, or other destructive work. | **Core safety convention** |
| `resolve-before-asking` | Exhausts brain and external evidence before asking the user to identify a person or relationship. | **On demand** |
| `gbrain-advisor` | Reports high-leverage brain improvements such as version drift, stalled jobs, or missing embeddings. | **Operator** |
| `skillpack-check` | Produces a compact health report combining doctor checks and pending migrations. | **Operator** |
| `gbrain-upgrade` | Responds safely to gbrain upgrade markers using the configured upgrade policy. | **Operator** |
| `smoke-test` | Runs post-restart health checks and known auto-fixes for gbrain environments. | **Operator** |
| `publish` | Renders a brain page into a shareable, optionally password-protected HTML artifact. | **On demand / adapter mismatch** |
| `brain-pdf` | Renders a brain page into a publication-quality PDF; the page remains the source of truth. | **On demand** |

### Creative thinking, research, and writing

| Skill | What it does | Alex fit |
|---|---|---|
| `idea-lineage` | Reconstructs one idea's first mention, best articulation, reversals, abandoned branches, and current version. | **Strong candidate** |
| `concept-synthesis` | Deduplicates raw concept pages, tiers them from canon to riff, clusters them, and builds an intellectual map. | **Background / dream-style** |
| `strategic-reading` | Applies a source to one strategic problem and produces a cited short-, medium-, and long-term playbook. | **Strong on demand** |
| `book-mirror` | Produces a chapter-by-chapter personal mirror of a book using brain context, without prescribing what the reader must do. | **Strong on demand** |
| `draft-in-voice` | Drafts in a validated person's voice using a brain-stored voice profile and a fidelity self-check; never auto-posts. | **Conditional** |
| `article-enrichment` | Converts raw article text into structured, quotable, cross-referenced brain pages. | **On demand / source-dependent** |
| `research-compendium` | Archives primary sources, summarizes each one, and synthesizes a reusable research asset. | **On demand / heavy** |
| `perplexity-research` | Combines brain context with current web research to distinguish new information from what the brain already knows. | **Conditional / external service** |
| `fact-check` | Verifies claims one by one against live citable sources and blocks unsupported output. | **Conditional** |
| `academic-verify` | Traces an academic claim through publication, method, data, and replication, then writes a cited result. | **Conditional** |
| `citation-graph-ingest` | Builds typed reference edges such as relies_on, overrules, or distinguishes across a corpus. | **Background / research** |
| `reports` | Saves and retrieves timestamped reports with keyword routing and link-quality checks. | **Conditional** |
| `cross-modal-review` | Uses a second model as a quality gate before committing or delivering important work. | **Optional quality gate** |
| `ask-user` | Presents two to four explicit choices and pauses until the user decides. | **Core convention** |
| `context-audit` | Audits always-loaded context for redundancy, contradictions, staleness, and token savings. It never edits. | **Operator-only** |

### Content and media ingestion

| Skill | What it does | Alex fit |
|---|---|---|
| `signal-detector` | Watches every substantive inbound message for original thinking and entity mentions, then captures them. | **Not default; policy conflict** |
| `idea-ingest` | Ingests a shared link, article, tweet, or idea, analyzes it, creates source/author context, and links it. | **On demand / external input** |
| `media-ingest` | Ingests video, audio, PDF, book, screenshot, or repository content with entity propagation. | **On demand / external input** |
| `voice-note-ingest` | Preserves exact voice-note phrasing while routing the note into appropriate brain pages. | **Conditional / no raw-transcript default** |
| `meeting-ingestion` | Normalizes meeting recordings or transcripts, enriches attendees, merges timelines, and verifies sequence. | **Out of scope for Alex** |
| `blog-ingest` | Ingests an entire blog, newsletter, or RSS/Atom publication with pagination, deduplication, and pacing. | **Background / external input** |
| `conversation-archive` | Imports AI chat exports and agent transcripts as dated conversation pages, then supports archive questions. | **Out of scope by policy** |
| `bulk-ingestion` | Runs a resumable, manifest-backed lifecycle for large corpora, from trial through monitoring. | **Background / operator** |
| `two-tier-extraction` | Uses utility, reasoning, and deep model tiers to triage and extract large corpora while controlling spend. | **Background / dream-style** |
| `data-research` | Researches structured data, archives raw sources, deduplicates, and maintains tracker pages from recipes. | **Conditional / domain-specific** |
| `archive-crawler` | Scans explicitly allow-listed personal archives for high-value writing, ideas, and relationships. | **Not now; high-scope input** |
| `webhook-transforms` | Converts external webhook events into sanitized, citable brain-ingest signals. | **Integration / not conversational** |

### Operations, personal-assistant, and setup workflows

| Skill | What it does | Alex fit |
|---|---|---|
| `setup` | Initializes gbrain, provisions PGLite or Supabase, injects agent rules, and imports initial data. | **Operator / setup only** |
| `cold-start` | Sequences the first high-leverage data imports for an empty brain. | **Operator / setup only** |
| `migrate` | Migrates Obsidian, Notion, Logseq, Markdown, CSV, JSON, or Roam data into gbrain. | **Operator / one-time** |
| `company-brainify` | Sanitizes a personal brain into a shared team/company brain and removes sensitive material. | **Out of scope** |
| `schema-author` | Proposes, adds, changes, or audits page types, aliases, prefixes, and link types in the active schema pack. | **Operator** |
| `schema-unify` | Migrates a noisy schema to the canonical gbrain-base-v2 taxonomy through an assessed Minion workflow. | **Operator / migration** |
| `minion-orchestrator` | Submits, monitors, steers, pauses, resumes, and replays durable background jobs and subagents. | **Background / operator** |
| `cron-scheduler` | Configures schedules, staggering, quiet hours, and wake-up overrides. | **Background / operator** |
| `briefing` | Compiles daily briefings with meetings, active deals, and citation tracking. | **Out of scope** |
| `daily-task-manager` | Maintains a stable-ID task lifecycle in brain pages. | **Out of scope** |
| `daily-task-prep` | Prepares the day from calendar context, meetings, open threads, and tasks. | **Out of scope** |
| `measure-before-you-fix` | Measures stale, slow, wedged, or timed-out operations before changing thresholds or architecture. | **Operator** |
| `eiirp` | Audits significant work, files durable outputs, checks schema/skill health, and reports what remains. | **Optional post-work; use sparingly** |
| `functional-area-resolver` | Compresses large resolver files into functional-area dispatchers without losing reachability. | **Meta / operator** |
| `soul-audit` | Runs the bootstrap identity interview and re-renders generated identity files. | **Out of scope; custom identity** |

### Skill-system and repository meta-work

| Skill | What it does | Alex fit |
|---|---|---|
| `skill-creator` | Creates a new conformant skill, checks overlap, updates the manifest/resolver, and validates it. | **Meta / operator** |
| `skillify` | Turns a repeated feature into a tested, routable, eval-backed skill with a no-regression quality bar. | **Meta / operator** |
| `skill-optimizer` | Optimizes one skill's body against a benchmark with validation gates and atomic writes. | **Meta / operator** |
| `skill-autobench` | Mines real usage and corrections to create a benchmark for an existing skill without rewriting it. | **Meta / operator** |
| `skillpack-harvest` | Generalizes a proven host skill and lifts it back into the reusable gbrain bundle. | **Meta / operator** |
| `testing` | Validates skill frontmatter, resolver coverage, manifest membership, conformance, and test health. | **Repository operator** |

## Supporting files that are not standalone skills

These files shape the skill system but should not be added as Pi skills:

- `skills/RESOLVER.md` — human-readable trigger routing and disambiguation.
- `skills/_AGENT_README.md` — skill discovery, invocation, update, and privacy
  contract.
- `skills/_brain-filing-rules.md` / `.json` — page filing and taxonomy rules.
- `skills/_output-rules.md` — output quality standards.
- `skills/conventions/` — shared brain-first, quality, routing, schema, safety,
  and subagent conventions.
- `skills/migrations/` — versioned migration notes, not conversational skills.
- `skills/manifest.json` — machine-readable skill inventory.
- `skills/skills.lock.json` — generated integrity/version lock.

## Portability note

The skill files are markdown recipes, not a universal executable API. When
moving them to another agent, first compare the skill's declared `tools:` with
the target harness's actual tools. A recipe that declares `query` or
`add_link` needs either those tools, a native CLI path, or an adapter
translation. Keep the skill on disk and route it on demand rather than
pretending unavailable tools exist.

At this checkout, `skills/manifest.json` lists all 69 routed skills while the
plugin bundle lists 61; the remaining entries are available in the repository
but are not included in that plugin bundle. The per-skill frontmatter and body
are the authority if this summary drifts.
