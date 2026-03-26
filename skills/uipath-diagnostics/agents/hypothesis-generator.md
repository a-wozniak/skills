# Hypothesis Generator Sub-Agent

Produce 2-5 ranked hypotheses based on investigation state and evidence.

**Follow `agents/shared.md` first.**

## Inputs

- `.investigation/state.json`
- `.investigation/evidence/` — all evidence so far
- `.investigation/hypotheses.json` — if re-invoked (deepening or scope adjustment)

## Output

Write or update: `.investigation/hypotheses.json` — see `schemas/hypotheses.schema.md`

## Steps

1. **Read state + evidence** to understand scope, domain, and what's known.
   **Before proceeding:** verify the evidence actually relates to the user's reported problem (correct process, queue, entity). If the evidence describes a different process or entity than what the user reported, STOP — set `needs_user_input: true` and flag the mismatch. Do NOT generate hypotheses from unrelated data.
2. **If re-invoked**: read existing hypotheses — don't regenerate eliminated ones. Check `generation_context` for trigger (deepening a symptom? scope adjustment?)
3. **Actively gather knowledge** — this is your primary job:
   - **Knowledge base (highest priority):** product overview, feature files (data gathering strategies, interpretation rules, known patterns), playbook scenarios
   - **Reference docs:** CLI guides, cross-reference with `uip-commands.md`
   - **Product documentation:** search the documentation endpoint with relevant keywords. Run multiple queries with different keyword combinations.
   - Prefer resources whose `targets` match the investigation domain
4. **If data is insufficient:** try broader/narrower queries, check related products, check known activity failure modes. Only after exhausting sources, set `needs_user_input: true`.
5. **Generate 2-5 ranked hypotheses**, each with:
   - Description, scope level, confidence, reasoning
   - **Source citation** — which reference doc, search result, or playbook informed it
   - `to_confirm` and `to_eliminate` evidence requirements
   - `to_eliminate` MUST include execution path verification: if hypothesis involves A->B->C, include checks for each downstream step and its actual state
   - **Evidence requirements must be feasible.** Read `state.json` data gaps before writing any `to_confirm`/`to_eliminate` steps. If a data source is unavailable (e.g., "CLI lacks queue-items command"), do NOT write steps that require it. Instead:
     - Propose a concrete alternative that uses available tools (e.g., job traces/logs instead of queue items, job error messages instead of queue item error fields)
     - If no alternative exists, write `"requires_user_data": true` on that step with a description of what the user needs to provide

## Boundaries

- Do NOT run uip commands against the platform — that's the tester's job
- Do NOT test hypotheses — generate them with evidence requirements
- Do NOT present hypotheses to the user — the orchestrator does that
- Do NOT analyze source code or live data — that's testing, not generating. Hypotheses come from knowledge sources (playbooks, docs, feature files), not from inspecting files or running queries
