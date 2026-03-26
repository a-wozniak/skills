---
product: orchestrator
scenario: Trigger stopped firing and audit shows system auto-disabled it
level: product
---

# Trigger Auto-Disabled

## Symptoms

- Trigger stops firing — no new jobs launched
- Audit log shows "System Administrator deactivated trigger"
- Queue items accumulate without processing

## Triage

- Check audit log for the trigger disable event
- Get the trigger details and confirm it is disabled

## Hypothesis Generation

- 10 consecutive failed launches within 24 hours triggered auto-disable
- No available robots when trigger evaluated
- License exhaustion — not enough unattended licenses
- Process errors — the process itself crashes on startup

## Testing

- Check robot availability in the folder at the time of failures
- Check license consumption: are all unattended licenses in use?
- Check recent job history for the associated process — are jobs faulting immediately?
- For queue triggers: verify the queue exists and has items

## Resolution

- Fix the underlying cause (robot availability, licensing, process errors)
- Re-enable the trigger:
  ```
  ProcessSchedules_SetEnabled(FolderId, TriggerId, true)
  ```
- Monitor for the next 24 hours to ensure it doesn't auto-disable again
