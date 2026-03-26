# Shared Agent Instructions

All diagnostic sub-agents follow these rules.

## Startup

1. Read `resources.yaml` to discover available tools, endpoints, and reference paths
2. Create `.investigation/`, `.investigation/evidence/`, `.investigation/raw/` if they don't exist

## Raw Data Rule

- Write full raw responses to `.investigation/raw/` **immediately**
- Do NOT keep raw data in context — write first, read back only specific fields if needed
- Evidence files reference raw files via `raw_data_ref`

## Playbook Discovery

1. Read `{type}/{product}/overview.md` for product context
2. Scan `{type}/{product}/playbooks/**/*.md` — match `scenario` frontmatter to symptoms
3. Filter by version: `(since is null OR version >= since) AND (until is null OR version <= until)`
4. Read any playbooks listed in `inherits` frontmatter
5. For activity-level failures: read `activity-packages/index.md`, find matching package, scan its playbooks
6. Read feature files (`features/*.md`) for data gathering strategies and interpretation rules

## uip CLI Usage

- Discover subcommands first: `uip --help` or `uip <subcommand> --help`
- Always use `--format json` for structured output if available
- Use `-o, --output <path>` when available to write directly to `.investigation/raw/`
- If a needed command doesn't exist, note the gap — do NOT fall back to REST/curl

## Data Integrity — CRITICAL

**Only work with verified, relevant data.** Every piece of data you use must be confirmed to relate to the user's reported problem (correct process, correct queue, correct time window, etc.). If a query returns data, check that it matches the reported context before using it.

**If you cannot retrieve the data you need: STOP.**
- Set `needs_user_input: true`, explain what data is missing and why you can't get it.
- Do NOT use unrelated data as a "proxy" or substitute.
- Do NOT infer, guess, or fabricate what the data might contain.
- Do NOT pivot to analyzing whatever data happens to be available — that leads to investigating the wrong problem.
- An honest "I can't get this data" is always better than a confident investigation built on wrong data.

## Constraints

- Do NOT generate or execute code (no Python scripts, no inline code). Shell commands for file I/O and uip are fine.
- Do NOT perform work outside your role (see your agent file for boundaries)
- If you need user input: set `needs_user_input: true` and `user_question` in your output, then stop

## Output Schemas

See `schemas/` for the canonical JSON schemas: `state.schema.md`, `hypotheses.schema.md`, `evidence.schema.md`.
