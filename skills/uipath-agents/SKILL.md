---
name: uipath-agents
description: UiPath agent lifecycle assistant. Works with both coded agents (Python with LangGraph/LlamaIndex/OpenAI Agents) and low-code agents (agent.json from Agent Builder). Orchestrates setup, auth, build, run, evaluate, deploy, and sync. Use when the user wants to create, run, evaluate, or deploy a UiPath agent, e.g. "create a UiPath agent", "set up and deploy an agent", "edit my agent.json", or "build and run an agent end-to-end".
allowed-tools: Bash, Read, Write, Glob, Grep, AskUserQuestion
user-invocable: true
---

# UiPath Agents

## Agent Type Detection

Before starting, determine the agent type. The CLI auto-detects at runtime, but the **build step** differs:

| Signal | Agent Type |
|--------|-----------|
| User mentions Python, LangGraph, LlamaIndex, OpenAI Agents, or writing code | **Coded Agent** |
| User mentions agent.json, Agent Builder, low-code, visual design, or no-code | **Low-Code Agent** |
| Project already contains `agent.json` (no `main.py` / `langgraph.json`) | **Low-Code Agent** |
| Project already contains `main.py`, `langgraph.json`, `uipath.json` with functions | **Coded Agent** |
| Ambiguous | **Ask the user** |

**Key insight:** Both agent types use the **same CLI commands** for run, eval, deploy, push, and pull. The only differences are in **setup** and **build**.

## CLI Setup (run once at start)

```bash
# 1. Check uip is installed
which uip > /dev/null 2>&1 && echo "uip found" || echo "uip NOT found — run: npm install -g @uipath/cli"

# 2. Activate the virtual environment (required if .venv exists)
if [ -d ".venv" ]; then source .venv/bin/activate; fi

# 3. Set up the Python environment (required for both agent types — the runtime is Python)
uip codedagents setup --format json
```

**Steps 2 and 3 are required for both coded and low-code agents.** The runtime that executes low-code agents is still Python under the hood.

If `uip` is not found, install it with `npm install -g @uipath/cli`. If `npm` is missing, ask the user to install Node.js first.

**Do NOT add `--format json` to forwarded commands.** The `--format` flag is only valid for native `uip` commands (like `uip login`, `uip codedagents setup`). Commands forwarded to the Python CLI (`new`, `init`, `run`, `eval`, `deploy`, `push`, `pull`, `pack`, `publish`, `invoke`) do **not** accept `--format json`.

## Critical Rules (Both Agent Types)

- **NEVER run `uip login` without `--tenant`.** The interactive tenant picker does not work from Claude's Bash tool. Always ask the user for environment, organization, and tenant name first.
- **Skip auth if already authenticated.** Check if `.env` contains `UIPATH_URL` and `UIPATH_ACCESS_TOKEN`. If yes, skip auth.
- **Auth MUST be an interactive question (when needed).** Output ONLY this question as your entire response:
  > What is your UiPath **environment** (cloud/staging/alpha), **organization name**, and **tenant name**?
- **Always create a smoke evaluation set.** Every agent must include `evaluations/eval-sets/smoke-test.json` with 2-3 basic test cases.

### Coded Agent Rules (Python only)

- **NEVER add a `[build-system]` section to `pyproject.toml`**. No `hatchling`, no `setuptools`. Only `[project]`, `[dependency-groups]`, and `[tool.*]` sections.
- **Select a framework before writing any code.** If ambiguous, ask the user to choose from: Simple Function, LangGraph, LlamaIndex, or OpenAI Agents.
- **Correct SDK import: `from uipath.platform import UiPath`** — not `from uipath import UiPath`.
- **Always use lazy LLM initialization.** Never instantiate LLM clients at module level — `uip codedagents init` imports the file and module-level clients will fail.

### Low-Code Agent Rules (agent.json only)

- **No `pyproject.toml` needed.** Low-code agents don't use Python packaging.
- **No `uipath init` needed.** The schema is already in `agent.json`.
- **No framework selection.** Low-code uses UiPath's built-in ReAct engine.
- **The entrypoint is always `agent.json`.** Use it in place of `main` or other named entrypoints.

## Lifecycle Stages

Both agent types share the same lifecycle. Stages marked with 🔀 differ by agent type.

| Stage | Reference | CLI Commands |
|-------|-----------|-------------|
| 🔀 **Setup** | Coded: [lifecycle/setup.md](references/lifecycle/setup.md) · Low-code: [lowcode/setup.md](references/lowcode/setup.md) | `uip codedagents new <name>` (coded) · Create `agent.json` (low-code) |
| 🔀 **Build** | Coded: [lifecycle/build.md](references/lifecycle/build.md) · Low-code: [lowcode/agent-json-reference.md](references/lowcode/agent-json-reference.md) | Write Python code (coded) · Edit `agent.json` (low-code) |
| **Auth** | [lifecycle/authentication.md](references/lifecycle/authentication.md) | `uip login` |
| **Bindings** | Coded: [lifecycle/bindings-reference.md](references/lifecycle/bindings-reference.md) · Low-code: [lowcode/resources-reference.md](references/lowcode/resources-reference.md) | Sync `bindings.json` (coded) · Add resources to `agent.json` (low-code) |
| **Run** | [lifecycle/running-agents.md](references/lifecycle/running-agents.md) | `uip codedagents run <ENTRYPOINT> '<input>'` |
| **Evaluate** | [lifecycle/evaluate.md](references/lifecycle/evaluate.md) | `uip codedagents eval <ENTRYPOINT> <eval-set>` |
| **Deploy** | [lifecycle/deployment.md](references/lifecycle/deployment.md) | `uip codedagents deploy --my-workspace` |
| **Sync** | [lifecycle/file-sync.md](references/lifecycle/file-sync.md) | `uip codedagents push`, `uip codedagents pull` |

**Entrypoint note:** For coded agents, `<ENTRYPOINT>` is the name from `entry-points.json` (e.g., `main`). For low-code agents, it is always `agent.json`.

## One-Prompt Flow

When the user asks to create and deploy an agent end-to-end, follow these steps in order. Skip stages that are already done.

**IMPORTANT: Do NOT stop between steps to ask "would you like me to continue?". Execute the entire flow automatically. Only pause when you genuinely need information from the user (auth credentials, project ID).**

### Step 1 — Detect Agent Type

Determine coded vs low-code from context (see Agent Type Detection table above). If ambiguous, ask.

---

### Steps 2-3 — Setup & Build (branched by agent type)

#### If Coded Agent:

1. **Framework** — Select from: Simple Function, LangGraph, LlamaIndex, or OpenAI Agents. Tell the user which you selected and why.
2. **Setup** — Scaffold: `uv add uipath-langchain` (or equivalent), `uv sync`, `uip codedagents new <name>`, `uip codedagents init`.
3. **Build** — Implement agent logic using the framework's patterns. Re-run `uip codedagents init` after changing Input/Output models. Read the [build reference](references/lifecycle/build.md) for framework-specific guidance.
4. **Bindings** — If using platform resources, sync `bindings.json` per [bindings-reference.md](references/lifecycle/bindings-reference.md).

#### If Low-Code Agent:

1. **Setup** — Create `agent.json` using the template from [assets/templates/agent.json](assets/templates/agent.json) or pull from Studio Web. Read the [low-code setup guide](references/lowcode/setup.md).
2. **Build** — Edit `agent.json`: configure prompts (`messages`), LLM settings (`settings`), input/output schemas, and add resources (tools, contexts, escalations). Read the [agent.json reference](references/lowcode/agent-json-reference.md) and [resources reference](references/lowcode/resources-reference.md).

---

### Steps 4-9 — Shared Lifecycle (identical for both types)

4. **Auth** — Check if `.env` already has `UIPATH_URL` and auth tokens. If not, ask:
   > What is your UiPath **environment** (cloud/staging/alpha), **organization name**, and **tenant name**?
   Then run `uip login --format json` followed by `uip login tenant set "<TENANT>" --format json`.

5. **Run** — Test locally:
   ```bash
   # Coded agent:
   uip codedagents run main '{"query": "test"}'
   # Low-code agent:
   uip codedagents run agent.json '{"task": "test"}'
   ```

6. **Push** — Tell the user to go to Studio Web (`{UIPATH_URL without tenant}/studio_/projects`), create a new project (Coded Agent or Agent project), and paste the project ID. Add `UIPATH_PROJECT_ID=<id>` to `.env`, then run `uip codedagents push`.

7. **Evaluate** — Create evaluator config and eval set, then run:

   Create `evaluations/evaluators/llm-judge-trajectory.json`:
   ```json
   {
     "version": "1.0",
     "id": "LLMJudgeTrajectoryEvaluator",
     "evaluatorTypeId": "uipath-llm-judge-trajectory-similarity",
     "evaluatorConfig": {
       "name": "LLMJudgeTrajectoryEvaluator",
       "defaultEvaluationCriteria": {
         "expectedAgentBehavior": "Agent should process the input and return a response."
       }
     }
   }
   ```

   Create `evaluations/eval-sets/smoke-test.json` with 2-3 test cases matching the agent's input schema:
   ```json
   {
     "version": "1.0",
     "id": "smoke-test",
     "name": "Smoke Test",
     "evaluatorRefs": ["LLMJudgeTrajectoryEvaluator"],
     "evaluations": [
       {
         "id": "test-1",
         "name": "Basic test",
         "inputs": {"field": "value"},
         "evaluationCriterias": {
           "LLMJudgeTrajectoryEvaluator": {
             "expectedAgentBehavior": "Agent should process the input and return a response."
           }
         }
       }
     ]
   }
   ```

   Run evals:
   ```bash
   # Coded: uip codedagents eval main evaluations/eval-sets/smoke-test.json
   # Low-code: uip codedagents eval agent.json evaluations/eval-sets/smoke-test.json
   ```

8. **Deploy** — Run `uip codedagents deploy --my-workspace`. For coded agents, bump patch version in `pyproject.toml` if re-deploying.

## Framework Selection (Coded Agents Only)

1. **Simple Function** — Plain Python with `Input`/`Output` models. No LLM. Best for deterministic logic.
2. **LangGraph** — StateGraph with conditional routing, tool use, interrupts. Best for complex LLM agents.
3. **LlamaIndex** — Workflow with events and RAG support. Best for knowledge retrieval.
4. **OpenAI Agents** — Lightweight agent with tools and handoffs. Best for simple LLM agents.

**Inference hints:** tools/multi-step → LangGraph. RAG → LlamaIndex. Simple LLM → OpenAI Agents. No LLM → Simple Function.

## Troubleshooting

| Error | Agent Type | Cause | Solution |
|-------|-----------|-------|----------|
| `Project authors cannot be empty` | Coded | Missing `authors` in `pyproject.toml` | Add `authors = [{ name = "Your Name" }]` |
| `Version already exists` on deploy | Coded | Same version published | Bump patch version in `pyproject.toml` |
| `Your local version is behind...Aborted!` | Both | Push needs confirmation | Use `uip codedagents push --overwrite` |
| `agent.json not found` | Low-code | Missing agent definition | Create `agent.json` per [setup guide](references/lowcode/setup.md) |
| `agent.json failed schema validation` | Low-code | Invalid JSON structure | Check against [agent.json reference](references/lowcode/agent-json-reference.md) |

## Resources

- **UiPath Python SDK**: https://uipath.github.io/uipath-python/
- **UiPath Evaluations**: https://uipath.github.io/uipath-python/eval/
- **agent.json Reference**: [references/lowcode/agent-json-reference.md](references/lowcode/agent-json-reference.md)
- **Resources Reference**: [references/lowcode/resources-reference.md](references/lowcode/resources-reference.md)
