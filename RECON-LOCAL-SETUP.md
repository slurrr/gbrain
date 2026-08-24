# RECON: gbrain as personal agent — preflight overview

**Execution state (2026-08-20):** workspace live at `~/Alex-workspace` (git, main,
commit cae52a5). Alex's 8-file stack ported verbatim from
`~/code/dev/pi-collab/agents/Alex/`. gbrain 0.46.24.0 installed (bun global,
`~/.local/bin/gbrain`). Brain: PGLite `~/.gbrain/brain.pglite`, keyless
embeddings, `search.mode=conservative`. Local LLM routed: tabbyAPI
`http://127.0.0.1:8002/v1` (`openai:qw3.8-3.67-direct`) for extraction/synthesis/
subagents. Interview: all 12 keys transcribed from the stack, UNCONFIRMED
(hash `c337cdd8…`) — human read-back + `--confirm` is the next gate. Reranker:
intentionally off until a server exists. Embeddings: wait for the embedding
model to be loaded in tabbyAPI.

Personal recon note (untracked, not for upstream). Synthesizes README /
BOOTSTRAP_FOR_AGENTS.md / docs/guides/bootstrap.md / AGENT_BOOTSTRAP_DESIGN.md /
topologies.md / system-of-record.md, expanded on the Codex limitations and the
planned local path: **pi as harness + locally hosted models on an
OpenAI-compatible endpoint.**

## The model in one paragraph

Open a harness in a new empty folder, paste the bootstrap block. The folder
becomes the agent's body: a private GitHub repo holding identity files
(SOUL/USER/MEMORY/AGENTS/CLAUDE/HEARTBEAT/ACCESS_POLICY/GITHUB, rendered only
from interview answers), `brain/` (registered as a gbrain source), `memory/`,
`skills/`, `state/`, plus an `agent.json` manifest. The PGLite DB lives in
`~/.gbrain` and is a **derived cache** — the repo's markdown is the system of
record (CI-enforced). Wipe the DB, `gbrain sync && extract all`, done. Second
machine = clone + `gbrain bootstrap attach`. Repo-as-body is the strong part:
ownable, inspectable, deletable, portable across harnesses (`format_version: 1`).

## Codex limitation, expanded

Question: "open any repo on this machine and it's automatic? only with that
agent?"

- **Scope:** `codex mcp add` has no scope flag. Bootstrap writes the gbrain
  MCP server into the **user-global Codex config** (`~/.codex/config.toml`).
  Every Codex session on this machine — any folder, any repo — spawns the
  server and exposes its read+write tools. It is Codex-only: Claude Code, pi,
  Cursor etc. are unaffected.
- **"Automatic":** the *tools* are exposed in every Codex session, yes. The
  brain-first *behavior* is not — the protocol lives in the agent workspace's
  AGENTS.md. In any other repo the brain is reachable (read **and** write) but
  ungoverned: a coding session in an unrelated project can silently read the
  personal brain, or worse, write stray pages into it. That is the real risk,
  and the docs state it plainly rather than solving it.
- **No per-turn push.** Codex gets pull-based context only (AGENTS.md gates +
  mandated MCP writes). Codex 0.147+ ships a hook system; gbrain does not wire
  it yet. No session-end push, no compaction checkpoints.
- **Model is cloud OpenAI** (subscription) — the opposite of the local plan.
- **Off-ramps:** `codex mcp remove gbrain` (registration only) or
  `gbrain bootstrap uninstall` (full teardown).

Net: Codex is the paved road, but on a shared machine its user-global MCP is a
genuine exposure, and it is the wrong model story for this setup.

## Codex vs local (pi + local OpenAI-compatible endpoint)

| | Codex path | pi + local path (plan) |
|---|---|---|
| Bootstrap | Paved: paste block, interview→render→repo→verify, ~15 min | Manual: same CLI phases run by hand; `hooks --harness` has no pi target |
| Per-turn context | None (pull protocol only) | DIY: pi extension `before_agent_start` injects brain context each turn |
| Persistence | Pull + mandated writes; no per-turn push | DIY: `turn_end`/`agent_settled` → debounced scan-gated push; `session_start`/`session_shutdown` |
| Brain reach | User-global — every Codex session on the machine | Project-scoped (`.pi/`) or user-scoped (`~/.pi/`) — our choice |
| Model | OpenAI cloud, subscription | Local endpoint, $0/token, quality-dependent |
| Main risk | Brain exposed to every repo opened in Codex | Local-model extraction quality gets baked into the system of record |

### Local model wiring (verified against repo docs)

- `OPENAI_BASE_URL=http://<host>:<port>` + `OPENAI_API_KEY=<anything>` points
  the native OpenAI provider (chat/expansion **and** embeddings) at the local
  endpoint; a bare host is auto-normalized to carry `/v1`.
  (`docs/integrations/embedding-providers.md`)
- LLM routing for `think`/synthesis/fact-extraction/subagents:
  `gbrain config set models.default openai:<model>` and per-tier
  `models.tier.{utility,reasoning,deep,subagent}`.
- Embeddings: local server's `/embeddings` via
  `--embedding-model openai:<model> --embedding-dimensions N`, or the
  Ollama / llama-server / LiteLLM recipes (local, keyless).
- Reranker: off (`conservative`), or the `llama-server-reranker` recipe
  (Qwen3-Reranker via `llama-server --reranking`) for a local cross-encoder.
- Setting `OPENAI_API_KEY` makes gbrain treat the install as **keyed**
  (capability report: semantic search + auto fact extraction) — all traffic
  local, cost zero.

### Quality caveats (the real cost of "local")

- `gbrain think` synthesis and automatic fact extraction run on the local
  model, and **facts are FS-canonical** — bad extraction gets persisted into
  the markdown system of record, not just a cache. Kill switch:
  `gbrain config set facts.extraction_enabled false`; prefer agent-authored
  Facts fences early on.
- Subagent/minion durable jobs need a local model with reliable tool-calling;
  doctor warns on non-Anthropic subagent tiers (no prompt caching).
- Search-mode cost matrix is moot ($0), but the quality trade-offs (reranker
  on/off, mode) still apply.
- Single-writer PGLite still holds: one live session/serve per brain;
  concurrent pi sessions on the same brain contend.

## Pi path, concretely

1. pi is **not** a bootstrap-supported harness (claude-code / codex / opencode
   only). Run the phases manually in a pi session: `gbrain init --pglite` →
   `gbrain bootstrap interview` → `render` → `gbrain skillpack scaffold --all`
   → repo (create/adopt + privacy check) → `gbrain bootstrap verify` (expect
   harness-gap notes).
2. pi has **no native MCP by design** ("build CLI tools with READMEs, or an
   extension"). The 7 memory verbs: `recall` / `remember` / `entity` /
   `synthesize` / `forget` are first-class CLI commands; `context_pack` /
   `delta` are MCP-only (reachable via `gbrain call`). Brain-first protocol
   goes in the workspace AGENTS.md, which pi reads natively. Skillpack
   markdown is tool-agnostic and pi has native skills.
3. Optional but recommended: a small pi extension reproducing the Claude Code
   hook lane — `before_agent_start` (per-turn brain context), `turn_end` /
   `agent_settled` (debounced push), `session_start` / `session_shutdown`
   (session persistence, checkpoint at compaction). This is the main DIY
   piece; pi's extension API covers all of it.
4. Placement per machine map: workspace repo under `~/code/dev/`, brain DB in
   `~/.gbrain` (tool convention — leave it), no new top-level homes.

## Follow-up recon (session 2)

**What the two MCP-only verbs are actually for.** `context_pack` = session-start
warm-up (entity cards + open threads + hot facts, zero-LLM, sub-second);
`delta` = "what changed since my last cursor" (O(changes), per-session cursor,
7-day expiry). Codex's pull protocol uses exactly these two as its ambient-recall
seam (`docs/mcp/CODEX.md`). Without them (CLI-only pi): approximate with
`gbrain entity` + `gbrain recall` at session start, and `git log`/`git diff` on
the brain repo for "what changed" (the push pipeline commits everything, so git
history *is* the delta). Lost: one-call budgeted digest + cursor bookkeeping —
ergonomics, not capability. **If the pi-mcp extension is acceptable, adding it
project-scoped (`.pi/` in the agent repo only) is the cleanest option**: full
verb surface, project-scoped (no other-repo exposure — strictly better than
Codex's global), and the hooks extension is the only remaining DIY piece.

**Concurrency.** One live serve per brain (PGLite single-writer): with the
global Codex install, every Codex session *sees* the tools, but only one can
use the brain at a time — the second fails politely with a lock error. That
matches the desired one-session-at-a-time model. **Two brains ⇒ true
parallelism**: a Codex session and a pi session run concurrently against
separate DBs.

**Separate brains per harness: yes.** `GBRAIN_HOME` selects the active brain
(documented Topology 3 pattern). Codex: `codex mcp add` has no env flag in the
local stdio form, so give the spawned serve a wrapper
(`~/.local/bin/gbrain-codex-serve` exporting `GBRAIN_HOME` + `exec gbrain serve`);
pi: project-scoped extension/CLI wrapper sets its own. Consequence: separate
memories; sharing is manual (source federation/mounts, latent-space only).

**Local model for the memory layer under Codex: yes, and it's not a minion.**
All gbrain-internal LLM work is config-routed independently of the harness:
`models.default` / `models.tier.*` / `facts.extraction_model` / `GBRAIN_MODEL`
→ `openai:<local-model>` via `OPENAI_BASE_URL`. Codex (subscription) does the
conversation; embeddings, fact extraction, `think` synthesis, and subagent
loops run local at $0. Two distinct mechanisms:
1. **Fact extraction** — automatic cycle phase on page writes (the default
   "local agent writes memories" path; not a minion).
2. **Minions/subagents** — explicit durable LLM tool loops on the job queue
   (`gbrain agent run`, dream-cycle synthesize/consolidate) on
   `models.tier.subagent`. Use this only if you want a scheduled background
curator. Needs a local model with reliable tool-calling; doctor warns on
   non-Anthropic subagent tiers (no prompt caching).
Note: "agent-authored memory" (harness model writing Facts fences, the keyless
pattern) already costs only the Codex subscription — the local routing saves
gbrain-side spend (embeddings/extraction/synthesis/rerank).

## Session 3 recon (scope reset: one main agent, one repo, pi only)

**Codex is a model here, not a harness.** User runs Codex-model through pi. No
`codex mcp add`, no `~/.codex/config.toml`, no global-config exposure — all
earlier Codex-harness notes are moot for this setup. gbrain wiring is 100% pi's
(project-scoped extension/CLI).

**The PGLite lock, answered.** It is not a gbrain policy — it is the engine's
own data-dir lock: PGLite is Postgres-in-WASM embedded in-process, and one
data dir admits one live instance (same postmaster-lock rule as real Postgres).
Nothing to "pass"; bypassing it corrupts the DB. The designed concurrency path
is the other engine: **Postgres + pgvector** (self-hosted container fits the
machine map: workspace `~/code/dev/<name>`, runtime `~/runs/<name>`). Many
sessions/serves coexist via pooled connections — that is the shared/multi-
machine topology. **The trade** (degradation matrix): per-turn hook injection
is PGLite-only today — the hook IPC is a unix socket in the PGLite data dir
(`src/core/context/resolve-ipc.ts`); the context *assembler* is engine-agnostic,
the delivery socket is not. So: PGLite = per-turn push + one session at a time;
Postgres = concurrent sessions + pull protocol (context_pack/delta still work
as MCP ops via pi-mcp if ever added). Given "many sessions at once" is the
hard requirement, Postgres + pull is the coherent pick; per-turn push is the
nice-to-have that drops.

**What Claude Code's per-turn injection actually is (verified in
`src/core/context/turn-context.ts`): NOT a query, NOT the reranker, zero LLM.**
Deterministic window-driven assembly from three sources: (1) reflex pointers —
entities *mentioned in the recent conversation window* → slug pointers (pattern
matching); (2) ≤3 confidence-gated volunteered pages, deduped; (3) hot-facts
cache (world-only visibility always). Wrapped in a "data, not instructions"
envelope, budgeted ≤8KB (Claude hook output cap), trimmed lowest-confidence
first. Vector search + reranker fire only when the agent explicitly calls
search/think. That is why it is cheap enough to run every turn.

**Harness seam comparison.** Claude Code = push (hooks: context injected into
every prompt + Stop/SessionEnd/PreCompact persistence). Codex/opencode = pull
(no wired hooks; AGENTS.md brain-first protocol + context_pack/delta at thread
start / post-compaction / heartbeat). pi = gbrain wires nothing, but pi's
extension events map 1:1 to Claude's hooks: `before_agent_start` ≈
UserPromptSubmit, `turn_end`/`agent_settled` ≈ Stop, `session_start`/
`session_shutdown` ≈ SessionStart/SessionEnd, custom-compaction ≈ PreCompact.
The hindsight-style extension pattern the user already runs is the right shape.

## Session 4 recon (porting an existing agent)

User already has a soul/identity for an agent built into pi-collab; wants the
main-agent workspace to retain it. Code-verified answers:

- **Render never overwrites existing files** (`render.ts`: `existsSync →
  skipped` unless `--force`, which backs up first). Port files first, render
  fills only what's missing.
- **The interview is a data-entry mechanism, not a ceremony.** Answers live in
  `state/interview.json`; render fills `{{TOKEN}}s` from them; verify checks
  (a) no unresolved tokens, (b) byte floors on SOUL/USER *scaled by answered
  count* ("floors catch skipped interviews, never pad"), (c) secret scan,
  repo privacy, round-trip. A ported, substantive soul clears the floors.
- **The 6 required keys** (all already answered inside an existing soul):
  `AGENT_NAME`, `PRINCIPAL_NAME`, `AGENT_PURPOSE`, `AGENT_TOP_JOBS`,
  `PRINCIPAL_CONTEXT`, `VOICE_REGISTER`. Consents (`SEARCH_MODE`, `MCP_SCOPE`,
  `PERSIST_CRON`, `HOOKS_CONSENT`, `PROVIDER_KEY`) are operational and still
  recorded fresh.
- **Port, don't re-interview.** Transcribe the 6 answers from the ported files
  (`interview --set`), read back, `--confirm <hash>` — the confirmation gate
  is preserved as a review of what's already written, not a fresh interrogation.
- **Identity vs operational contract split:** port SOUL/USER/MEMORY (identity);
  take the RENDERED AGENTS.md/CLAUDE.md/HEARTBEAT/ACCESS_POLICY/GITHUB
  (operational contract — carries the brain-first Gate-3 protocol + durability
  rules) and merge any existing operational rules into them. Don't let the old
  AGENTS.md swallow the brain protocol, or the rendered one overwrite kept rules.
- **Memory corpus ≠ MEMORY.md.** MEMORY.md is the session-start identity file;
  the existing agent's accumulated memory goes into `brain/` (import/sync) so
  it's searchable + graph-linked.
- **`gbrain bootstrap attach`** is the canonical existing-agent path, but for
  clones of an *already-bootstrapped* workspace (validates `agent.json`
  `initialized: true`). Hand-rolling a manifest for a non-bootstrap agent is
  possible but fragile; after the transcription path completes, the workspace
  IS a bootstrapped agent repo and attach works naturally on machine two — no
  interview, ever again.

## Verdict

The workspace-as-private-repo model is the right shape and the part worth
keeping regardless of harness. For this machine the pi+local path is actually
the *safer* one (no user-global brain exposure, model stays local) — it just
trades ~15 minutes of paved road for manual phases plus a ~100-line extension.
Codex remains the fallback if we ever want the paved road; accept its
user-global MCP or don't install it here.