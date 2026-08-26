# Current Goal
Run Alex as a custom Pi workspace using gbrain's memory-only integration. Alex's authored identity stays in the workspace; gbrain is the sole memory system. Pi remains the harness and AgentMux's local model is gbrain's backstop. No parallel filesystem memory, transcript capture, or bootstrap-generated agent body.

# Current State
- Workspace: `/home/poop/Alex-workspace`.
- Current commits: Alex workspace `d990f48 feat: integrate Alex with ambient gbrain memory`; gbrain fork `bf810f4bd docs: update Alex integration checkpoint`. Alex workspace now has new uncommitted tool-policy changes; gbrain fork is clean until this checkpoint refresh.
- Removed legacy `MEMORY.md` and `memory/`; these were Alex's old filesystem-memory layer, not gbrain components. `ideas/` and authored identity files remain preserved.
- `.gbrain-source` pins the workspace source to `workspace`; `gbrain sources current --json` resolves `source_id: workspace`.
- `.pi/extensions/gbrain.ts` registers and active-injects the core memory/page tools plus timeline and graph operations: recall, remember, entity, synthesize, context pack, delta, forget, page get/put, timeline read/write, links/backlinks read, link add/remove, graph traversal, and link-source listing. New `.pi/alex-policy.json` allowlists read/search plus all native gbrain tools; the bridge reapplies it after registration and before each turn, leaving skill discovery unchanged for now.
- All bridge calls use the native `gbrain call --source workspace` operation surface, run from the workspace root, and pass `OPENAI_BASE_URL=http://127.0.0.1:8002/v1`, local `OPENAI_API_KEY=dummy`, and `--quiet`. Page, fact, timeline, and link writes perform read-back verification.
- Ambient retrieval follows gbrain placement: confidence-gated `volunteer_context` per non-command turn; `context_pack` at session start and post-compaction; a session-cursor `delta` wake every eight turns. Full hybrid `recall` is explicit only. Trusted-local boundary/delta packs include private facts by deliberate creative-workspace policy. The rolling session window is used ephemerally for entity detection; no transcript archive or extraction is enabled.
- Creative-memory policy is project/collaboration/problem-solving focused: Alex's thinking preferences, how Alex and Seth work together, durable project facts, decisions/rationale, reusable workflows, and lessons. Schedules, email, errands, raw transcripts, ordinary brainstorm fragments, and unconfirmed speculation are out of scope. Remember visibility defaults explicitly to `world` so useful context is not filtered out.
- `.pi/extensions/agentmux.ts` discovers `/v1/models` at `http://127.0.0.1:8002/v1`, syncs the first advertised server id into gbrain `models.default`, and overlays only AgentMux endpoint/auth/compat settings in Pi. It deliberately does NOT provide a `models` array, preserving the curated `~/.pi/agent/models.json` local aliases. Workspace default is `agentmux/qw3.8-exl3`; `/alex-model` selects that alias. Pi-only fallback is `openai-codex/gpt-5.6-luna` at `xhigh`; gbrain-native work does not use that fallback.
- Pi extension commands load; `/alex prompt stats/json` showed the authored prompt stack in the intended order. With the endpoint currently serving only `qw3.8-3.67-direct`, Pi's stable `agentmux/qw3.8-exl3` alias successfully completed a real `OK` request and the response reported the canonical server model. `pi --list-models` now shows all eight curated AgentMux aliases plus Codex models. Bun builds and `git diff --check` pass. LSP still reports missing workspace dependencies/types; print-mode exits remain unverified because Pi hangs after settling when extensions are loaded.
- Runtime state: installed `gbrain` was upgraded to 0.46.30.0 and schema migration 141 applied; AgentMux port 8002 is down; full doctor reports stale workspace sync, one DB-only page with no backing file, no completed cycle, and no retrieval-reflex serve path. No brain reset/reindex/write was performed.

# Decisions
- Treat the current workspace as a custom Pi creative agent with gbrain memory-only integration, not `gbrain bootstrap`; custom identity/runtime ownership is correct.
- Keep no transcript capture/extraction for now. Gbrain's full bootstrap transcript/dream lane is a separate optional capability, not required for the memory-only protocol.
- Preserve native gbrain operations and use the documented ambient placement. The trusted local creative workspace intentionally widens context_pack/delta visibility and defaults ordinary remembered facts to world visibility; this is not a new privacy system.
- Do not expand fact/page verification or slug normalization yet; observe native gbrain behavior in real creative use first.
- Pi model discovery is health/gbrain routing only; the curated models.json catalog owns visible/selectable aliases. A discovered server id is not injected as a replacement model.

# Open Problems
- Decide whether to repair/export the orphan DB page and run `gbrain sync --source workspace`; do not delete it without approval.
- Validate the new ambient path with a real Pi turn: volunteer pointers, boundary pack after compaction, periodic delta, and explicit memory/page/graph tool round-trips. AgentMux is serving enough for tool-policy inspection, but gbrain model-backed operations remain untested end-to-end.
- The remaining doctor warnings about bootstrap receipt/serve/reflex state are old or expected custom-harness residue unless a live lock persists.
- Dream remains deferred and should be added only after the memory-only path passes a clean round-trip test.

# Resume Instructions
From `/home/poop/Alex-workspace`, inspect and commit the uncommitted `.pi/alex-policy.json` plus `gbrain.ts` tool allowlist when ready. Skills remain enabled and ungated for the next pass. Then test one non-command turn, boundary/compaction behavior, periodic delta, and one approved native page/fact/graph round-trip. Do not initialize, reset, reindex, or delete existing brain data without explicit approval.
