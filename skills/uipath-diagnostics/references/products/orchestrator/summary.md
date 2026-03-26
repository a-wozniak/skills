---
product: orchestrator
type: playbook-index
description: Summary of all known Orchestrator issues with links to detailed playbooks
---

# Orchestrator Playbooks Summary

| Issue | Scenario | Playbook |
|-------|----------|----------|
| Asset Not Found | Job fails with "GetAsset" exception or permission denied when reading an asset at runtime | [asset-not-found.md](./playbooks/asset-not-found.md) |
| Clock Skew in Cluster | Jobs triggered at wrong times or database faults in clustered/NLB environments due to unsynchronized clocks | [clock-skew.md](./playbooks/clock-skew.md) |
| Database Deadlock on Queue Statistics | Queue statistics hang or fail with performance degradation under high queue load (fixed in 2023.10.9) | [database-deadlock.md](./playbooks/database-deadlock.md) |
| Job Stuck in Running | Job remains in Running state indefinitely with no progress in traces | [job-stuck.md](./playbooks/job-stuck.md) |
| Orchestrator Down | Orchestrator completely inaccessible or returning 500 errors (IIS, config, or Identity redirect loop) | [orchestrator-down.md](./playbooks/orchestrator-down.md) |
| Queue Items Failing | Queue items transitioning to Failed status with various error types | [queue-items-failing.md](./playbooks/queue-items-failing.md) |
| Remote Certificate Invalid | Robots cannot connect due to TLS/SSL certificate validation failure | [remote-certificate-invalid.md](./playbooks/remote-certificate-invalid.md) |
| Robot Unresponsive | Robot marked as Unresponsive with heartbeat loss, jobs cannot be dispatched | [robot-unresponsive.md](./playbooks/robot-unresponsive.md) |
| Storage Bucket FileSystem Disabled | FileSystem provider not available when creating or configuring storage buckets | [storage-bucket-disabled.md](./playbooks/storage-bucket-disabled.md) |
| Trigger Auto-Disabled | Trigger stopped firing after 10 consecutive failed launches within 24 hours | [trigger-auto-disabled.md](./playbooks/trigger-auto-disabled.md) |
