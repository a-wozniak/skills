---
product: orchestrator
scenario: Job stuck in Running state and not completing
level: product
requirements:
  - id: folder_id
    scope: [process, feature, activity]
    prompt: Ask which Orchestrator folder the issue is in — most resources are folder-scoped
    auto_resolve: Folders_Get
    required: true
  - id: source_code_path
    scope: [process, activity]
    prompt: Ask for the project source code path — Orchestrator data shows WHAT failed but source code shows WHY
    auto_resolve: null
    required: true
    deferrable: true
    fallback_note: Fix location is approximate — source code was not available for review
---

# Job Stuck in Running

## Symptoms

- Job shows Running state for an unusually long time
- No progress visible in job traces
- Robot may still appear as Busy

## Triage

- Get the job details and check its runtime type
- Check if the job has a ParentJobKey — if so, query the parent job's state

## Hypothesis Generation

- Check job traces (GetJobTraces) for the last activity that executed
- Check if the process is a BPMN/Agentic Process (ProcessOrchestration runtime) — these have different stuck patterns than standard jobs
- For ProcessOrchestration jobs: check if the job is serverless (no robot/machine). If HostMachineName is empty and there's no robot session, the job is orchestration-only — heartbeat detection and robot-level timeouts don't apply
- Check if the job was started via a debug run — look for debug_overwrites.json in the source project. Debug runs can redirect folder bindings and change execution context

## Testing

- If the robot is unresponsive, check robot heartbeat status
- If the process is waiting for user input (attended), check if the Assistant dialog is visible
- If it's a ProcessOrchestration job, check if a child service task is stuck waiting for completion
