---
product: orchestrator
scenario: Queue statistics hang or fail with performance degradation under high queue load
level: product
until: "2023.10"
---

# Database Deadlock on Queue Statistics

## Symptoms

- Queue statistics page hangs or fails to load
- Performance degradation under high queue load
- Deadlock errors when querying queue-related database tables

## Triage

- Check Orchestrator version — this was fixed in 2023.10.9
- Check database logs for deadlock entries involving queue tables

## Testing

- Query SQL Server for recent deadlock events on Orchestrator database
- Check if the issue correlates with queue processing volume
- Verify Orchestrator version is below 2023.10.9

## Resolution

- Upgrade to Orchestrator 2023.10.9 or later (contains the fix for concurrent query deadlocks)
- If upgrade is not immediately possible: reduce concurrent queue processing load as a temporary workaround
