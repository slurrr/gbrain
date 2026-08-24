# Current Goal
Define the simplest non-lossy identity/runtime split for Alex: authored 00–70 stack through pi, gbrain memory intact, no duplicated identity.

# Current State
- Alex workspace: `/home/poop/Alex-workspace`; no files there were edited in this session.
- Current `.pi/SYSTEM.md` contains 00–70 + USER; native pi appends machine/workspace `AGENTS.md` and any native skills.
- Current `SOUL.md` is the 93-line gbrain interview/template synthesis and is not in the effective prompt. Current extension injects MEMORY.md once plus per-turn `gbrain search`; it does not capture transcripts.
- Installed gbrain 0.46.28 renderer verified: interview state resolves templates; existing files are skipped without `--force`; `--force` backs up then replaces; there is no merge or stack input. `soul-audit` explicitly force-renders, so it conflicts with an authored stack.
- Current workspace status reports bootstrap wire pending (expected: no supported pi harness); brain/repo/verify are done.

# Decisions
- Do not make gbrain bootstrap interview/render the ongoing identity source of truth.
- Preserve 00–70 as file-level authored source; use a workspace-owned deterministic view renderer for a complete SOUL layer.
- AGENTS should own operations/runtime/memory gates only; SOUL should own persona, judgment, role, and fidelity identity; USER should own Seth facts; SYSTEM should be a short pi runtime adapter.
- Keep MEMORY as gbrain extension message injection and keep recall injection; no transcript capture.

# Open Problems
- Choose static vs dynamic materialization of SOUL/USER into the effective system prompt. Dynamic extension loading best avoids regenerating SYSTEM; static generation is the lower-change alternative.
- Draft the non-lossy SOUL view and cleaned AGENTS, then verify exact effective prompt occurrence counts.
- Adapt/disable the copied `skills/soul-audit` path before identity edits so it cannot force-render away the stack.

# Resume Instructions
Read this checkpoint, then report the observed state and recommended architecture before editing `/home/poop/Alex-workspace`. If implementing: first make a deterministic stack→SOUL renderer, clean AGENTS, make `.pi/SYSTEM.md` runtime-only, and update `.pi/extensions/gbrain.ts` to prepend SOUL+USER as system instructions while retaining MEMORY/recall message injection. Verify with a throwaway pi extension using `ctx.getSystemPrompt()` and `gbrain bootstrap status --json`.
