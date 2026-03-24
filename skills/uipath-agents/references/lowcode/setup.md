# Low-Code Agent Setup Guide

This guide explains how to set up a UiPath low-code agent project for local development, testing, and deployment using the `uip` CLI.

---

## Prerequisites

Before you begin, ensure the following tools are installed:

- **Python 3.11 or higher** — The agent runtime is Python-based even for low-code agents. Python must be available on your `PATH`.
- **`uip` CLI** — The UiPath CLI used for pulling, running, evaluating, and publishing agents. Install or update it via:
  ```bash
  pip install uipath --upgrade
  ```
  Verify the installation:
  ```bash
  uip --version
  ```
- **UiPath account credentials** — You need access to a UiPath Automation Cloud tenant. Authentication tokens are stored in `.env`.

---

## Two Paths to Get an `agent.json`

Every low-code agent is defined by a single `agent.json` file. There are two ways to obtain one.

### Path 1: Pull from Studio Web (Recommended)

Use this path when you have already designed an agent in **UiPath Studio Web Agent Builder** and want to work with it locally.

1. Open Studio Web and navigate to your agent in Agent Builder.
2. Publish or save the agent so it has a registered identity in Automation Cloud.
3. In your terminal, create a local directory and pull the agent definition:
   ```bash
   mkdir my-agent && cd my-agent
   uip pull
   ```
   The CLI will prompt you to select the tenant, folder, and agent. It downloads `agent.json` and an optional `bindings.json` into the current directory.
4. Verify the files were created:
   ```bash
   ls -1
   # agent.json
   # bindings.json   (if the agent has deployed resource bindings)
   ```

> **Note:** `uipath pull` keeps your local `agent.json` in sync with Studio Web. Re-run it whenever the cloud definition changes.

---

### Path 2: Create `agent.json` from Scratch

Use this path when you want to author a brand-new agent entirely in code without using Studio Web first.

1. Create a project directory:
   ```bash
   mkdir my-agent && cd my-agent
   ```
2. Create `agent.json` manually. See the **Minimal Template** section below for a starting point, and consult [agent-json-reference.md](../agent-json-reference.md) for the full schema.
3. Optionally create a `.env` file with your credentials (see **Authentication** below).

---

## Minimal `agent.json` Template

The following is a minimal, working `agent.json` that you can copy and customise. All fields shown are required unless marked optional.

```json
{
  "$schema": "https://cloud.uipath.com/agent-schema/v1.1.0/agent-schema.json",
  "version": "1.1.0",
  "name": "My Agent",
  "description": "A short description of what this agent does.",
  "engine": "basic-v2",
  "model": "gpt-4o-2024-11-20",
  "prompts": {
    "system": "You are a helpful assistant. Answer the user's request clearly and concisely.",
    "user": "{{task}}"
  },
  "inputSchema": {
    "type": "object",
    "properties": {
      "task": {
        "type": "string",
        "description": "The task or question for the agent."
      }
    },
    "required": ["task"]
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "result": {
        "type": "string",
        "description": "The agent's response."
      }
    },
    "required": ["result"]
  },
  "resources": [],
  "features": []
}
```

For production agents with tools, context, or escalation, see [resources-reference.md](./resources-reference.md).

---

## No Virtual Environment Needed for Editing

Low-code agents do **not** require you to create or activate a Python virtual environment just to edit `agent.json`. You can open and modify the file in any editor directly.

However, a virtual environment **is** created automatically when you run the agent locally. The `uip codedagents setup` command bootstraps the Python runtime environment required to execute the agent:

```bash
uip codedagents setup
```

Run this once per project (or after updating dependencies). It creates a `.venv` directory in your project folder with all necessary runtime packages.

> You do **not** need to call `uip codedagents setup` merely to edit the agent definition or run evaluations — only when running the agent as a local server.

---

## No `uipath init` Needed

Low-code agent projects do **not** require `uipath init`. The `$schema` field inside `agent.json` already points to the official JSON Schema, which editors like VS Code use for validation and autocompletion. No separate initialisation step is required.

---

## No `pyproject.toml` Needed

Low-code agents do **not** use Python packaging. There is no `pyproject.toml`, no `setup.py`, and no need to define Python dependencies manually. The runtime dependencies are managed entirely by the `uip` toolchain.

If you see a `pyproject.toml` in a project, it likely belongs to a **coded agent** (Python-based). For low-code agents, ignore or delete it.

---

## No Framework Selection

Low-code agents use UiPath's built-in **ReAct engine** (`"engine": "basic-v2"`). There is no framework selection step and no need to choose between LangChain, LlamaIndex, or similar libraries. The orchestration logic is handled by the platform.

---

## Directory Structure

A complete low-code agent project looks like this:

```
my-agent/
├── agent.json              ← Agent definition (THE agent — edit this file)
├── bindings.json           ← Resource bindings (optional, used for deployed resources)
├── .env                    ← Authentication tokens (never commit to source control)
└── evaluations/            ← Evaluation datasets and custom evaluators
    ├── eval-sets/          ← YAML or JSON files defining test cases
    └── evaluators/         ← Custom evaluator scripts (optional)
```

### File Descriptions

| File / Directory | Purpose |
|---|---|
| `agent.json` | The complete agent definition: model, prompts, input/output schemas, resources, features. This is the only file required to define the agent. |
| `bindings.json` | Maps resource slugs to deployed UiPath assets (folders, processes, connectors). Generated by `uip pull`; not needed for authoring. |
| `.env` | Stores `UIPATH_URL`, `UIPATH_TOKEN`, and other secrets. Never commit this file. |
| `evaluations/eval-sets/` | Contains evaluation input/expected-output pairs used by `uip eval run`. |
| `evaluations/evaluators/` | Optional custom evaluator scripts that override default LLM-based scoring. |

---

## Authentication

Create a `.env` file in the project root with the following variables:

```dotenv
UIPATH_URL=https://cloud.uipath.com/{org}/{tenant}
UIPATH_TOKEN=<your-personal-access-token>
UIPATH_FOLDER_PATH=<your-orchestrator-folder>
```

You can generate a Personal Access Token in **UiPath Automation Cloud → Preferences → Privacy and Security**.

---

## Running the Agent Locally

Once `agent.json` is in place and `.env` is configured:

```bash
# First-time runtime setup (only needed once)
uip codedagents setup

# Run the agent with a test input
uip run agent.json --input '{"task": "Summarise the quarterly report."}'
```

---

## Publishing to Automation Cloud

```bash
uip publish
```

This packages and uploads the agent to your connected Orchestrator folder, making it available in Studio Web and for API invocation.

---

## Next Steps

- **Add tools and context:** See [resources-reference.md](./resources-reference.md) for all resource types.
- **Full schema reference:** See [agent-json-reference.md](../agent-json-reference.md) for every field.
- **Starter template:** Copy [../../assets/templates/agent.json](../../assets/templates/agent.json) as a project starting point.
