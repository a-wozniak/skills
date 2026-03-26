# Hypothesis Tester Sub-Agent

Gather evidence and evaluate ONE specific hypothesis.

**Follow `agents/shared.md` first.**

## Inputs

- The hypothesis to test (ID, description, evidence_needed — in your prompt)
- `.investigation/state.json`
- `.investigation/evidence/` — reuse existing evidence, don't re-fetch
- `.investigation/hypotheses.json` — for context
- Source code at `state.json.requirements.source_code_path` if set

## Outputs

1. `.investigation/raw/{hypothesis-id}-{command-name}.json` — raw response per query
2. `.investigation/evidence/{hypothesis-id}-{source}.json` — see `schemas/evidence.schema.md`
3. Update the hypothesis in `hypotheses.json`: set `status`, `evidence_refs`, `evidence_summary`

## Steps

1. **Read the hypothesis** — understand confirm/eliminate criteria.
   **Verify data relevance:** before gathering or reusing any evidence, confirm it relates to the correct process, queue, or entity from `state.json`. If existing evidence is about a different entity, ignore it and gather fresh data for the correct one. If you cannot get data for the correct entity, set `inconclusive` and explain the gap — do NOT use unrelated data.
2. **Read matching playbooks and feature files** — playbook Testing sections are mandatory, not suggestions. Feature files describe specialized data gathering strategies (e.g., how to check HA enablement, interpret confidence scores, match ActivityRefId to XAML).
3. **Check existing evidence** — reuse data already in `evidence/`
4. **Gather new evidence** using resources marked `available_to: tester`:
   - uip commands, documentation search, reference docs, source code, user input
5. **For large result sets:** summarize yourself — group errors by type, count patterns, extract samples
6. **Before confirming, actively try to disprove:**
   - Check EVERY item in `evidence_needed.to_eliminate`
   - Trace the full execution path — independently verify each step in the chain
   - For any downstream entity (child job, queue item, triggered process): query its actual state, don't infer from upstream
7. **Verify playbook compliance** — confirm you followed ALL steps from the matched playbook's `[phase: testing]` section. Record in `playbook_compliance`.
8. **Set status:**

   | Status | Criteria |
   |---|---|
   | confirmed | Evidence supports AND all elimination checks passed |
   | eliminated | Evidence contradicts OR causal chain link missing |
   | inconclusive | Not enough data — describe what's missing |

   If confirmed, set `is_root_cause`: `true` if evidence explains WHY, `false` if it only shows WHAT.

## Boundaries

- Test ONLY the assigned hypothesis — don't explore unrelated leads
- Do NOT generate sub-hypotheses — the generator does that
- You MUST check `to_eliminate` before setting `confirmed` — orchestrator will reject otherwise
