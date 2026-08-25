# Current Goal
Run Alex as a custom Pi workspace using gbrain's memory-only integration. Alex's authored identity stays in the workspace; gbrain is the sole memory system. Pi remains the harness and AgentMux's local model is gbrain's backstop. No parallel filesystem memory, transcript capture, or bootstrap-generated agent body.

# Current State
- Workspace: `/home/poop/Alex-workspace`.
- Latest commit: `cccfa1b remove custom facts handling from gbrain page bridge`; working tree is clean and this commit is one commit ahead of `origin/main`.
- Removed legacy `MEMORY.md` and `memory/`; these were Alex's old filesystem-memory layer, not gbrain components. `ideas/` and authored identity files remain preserved.
- `.gbrain-source` pins the workspace source to `workspace`; `gbrain sources current --json` resolves `source_id: workspace`.
- `.pi/extensions/gbrain.ts` registers and active-injects all ten Pi bridge tools: `gbrain_recall`, `gbrain_remember`, `gbrain_entity`, `gbrain_synthesize`, `gbrain_context_pack`, `gbrain_delta`, `gbrain_forget`, `gbrain_add_timeline_entry`, `gbrain_get_page`, and `gbrain_put_page`.
- The bridge tools invoke native gbrain CLI operations with `pi.exec`; `gbrain_forget` uses native `gbrain call forget`. They are retained so a creative agent can perform the exposed memory operations without Bash while Pi has no MCP.
- `gbrain_put_page` now delegates to native `gbrain put`, verifies with native `gbrain get`, and does NOT require the model to author a Facts fence or run a separate `extract_facts` phase. Gbrain owns Facts fences and reconciliation.
- Before each non-command agent turn, the extension runs native `gbrain recall --source workspace --query <prompt> --budget-tokens 2000 --json` and injects returned facts/results as data. It does not read filesystem memory or capture transcripts. It no longer calls stop/session-end persistence hooks.
- `.pi/extensions/agentmux.ts` discovers `/v1/models` at `http://127.0.0.1:8002/v1`, registers the local model with Pi, and syncs gbrain `models.default`. Pi-only fallback is `openai-codex/gpt-5.6-luna` at `xhigh`; gbrain-native work does not use that fallback.
- Bun extension build passes with Pi/typebox marked external; `git diff --check` passes. Known LSP diagnostics are missing global Pi/typebox/node declarations, not new runtime failures.
- Existing `/home/poop/.gbrain/brain.pglite` was not initialized, reset, reindexed, or written during this work. Autopilot/jobs and dream remain deferred. AgentMux on port 8002 is still down; no real local-model backstop test or approved real gbrain write has been performed.

# Decisions
- Use gbrain memory-only mode, not `gbrain bootstrap`; Pi owns identity/runtime and gbrain owns memory.
- Keep the ten Pi bridge tools and active-tool injection for the no-MCP Pi harness seam. Their underlying operations remain native CLI/gbrain calls; no second memory implementation exists.
- Remove only the page bridge's invented Facts policy: no manual fence validation, no model-authored-fence requirement, and no per-write `extract_facts` invocation.
- Treat `MEMORY.md`/`memory/` as legacy Alex filesystem memory, not as gbrain features. Their removal is part of making gbrain the sole memory system.
- No transcript pipeline, git push hooks, or dream/autopilot operation is enabled by preference/deferment.

# Open Problems
- Restart Pi so commit `cccfa1b` is loaded; verify all ten gbrain tools are active and a real pre-turn recall injection occurs.
- Start the approved AgentMux model on port 8002, run `/alex-model`, and compare `/v1/models`, Pi's selected model, and `gbrain config get models.default`.
- Perform one approved real `gbrain_remember` or `gbrain_put_page` write, verify native read-back/indexing, and separately validate the local model facts backstop when the worker/model path is available.

# Resume Instructions
From `/home/poop/Alex-workspace`, restart Pi to load `cccfa1b`. Confirm the ten gbrain bridge tools and active-tool injection, then send a non-command prompt and verify the hidden gbrain recall context. When port 8002 is serving, run `/alex-model`; compare `curl http://127.0.0.1:8002/v1/models` with Pi and `gbrain config get models.default`. Only after approval, perform one native gbrain write and read it back; do not initialize, reset, reindex, or alter the existing brain outside that approved test.
