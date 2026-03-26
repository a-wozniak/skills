---
product: orchestrator
scenario: Queue items failing with various error types across the queue
level: product
requirements:
  - id: folder_id
    scope: [process, feature, activity]
    prompt: Ask which Orchestrator folder the issue is in — most resources are folder-scoped
    auto_resolve: Folders_Get
    required: true
  - id: source_code_path
    scope: [process, activity]
    prompt: Ask for the project source code path (.xaml, .bpmn, project.json) — Orchestrator data shows WHAT failed but source code shows WHY
    auto_resolve: null
    required: true
    deferrable: true
    fallback_note: Fix location is approximate — source code was not available for review
---

# Queue Items Failing

## Symptoms

- Queue items transitioning to Failed status
- Multiple distinct error messages across items
- Successful items may still be processing normally

## Triage

- Identify which queue and folder are affected
- Get a count of total failed vs successful items

## Hypothesis Generation

- Compare input data (SpecificContent) between failed and successful items to identify data-driven failures
- Check if failures cluster around a specific time window or machine
- Check if the performer process was recently updated

## Testing

- Get ALL failed queue items (paginate if >100) — do NOT stop at the first page
- Categorize failures by error type — there may be MULTIPLE distinct failure modes
- After identifying a failure mode, count how many items it explains. If the count does not match the total number of failed items, there are additional failure modes — keep investigating until every failed item is accounted for
- Get 2-3 successful queue items for comparison
- Compare input data between failed and successful items

## Evaluation

- A confirmed hypothesis is only a complete root cause if it accounts for ALL failed items in the queue. If it explains only a subset (e.g., 156 of 160), the remaining items have a different root cause — classify the finding as partial and continue investigating
- When source code is available, trace the full execution path for EACH error category, not just the first one found. Different queue item data can trigger different code paths and different failures

## Shortcuts

### GetAsset Exception

- **Match**: error message contains "GetAsset" or "Asset not found" or "does not have the required permissions to access the asset"
- **Root cause**: the asset does not exist in the folder where the process is running, or the robot/user does not have permission to access it
- **Fix**: create the asset in the correct folder, or grant the robot account permission to access the asset. Check that the asset name matches exactly (case-sensitive) and that it exists in the same folder (or a parent folder with inheritance enabled) where the job runs
- **Still test**: yes — verify the asset name in source code matches what's configured in Orchestrator
