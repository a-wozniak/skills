---
product: orchestrator
scenario: Jobs triggered at wrong times or database state faults in clustered/NLB environments
level: product
---

# Clock Skew in Cluster

## Symptoms

- Jobs triggered at wrong times
- Database enters faulted state requiring IIS restart
- SQL database errors in NLB/clustered environments

## Triage

- Check if the deployment is clustered (multiple Orchestrator nodes behind a load balancer)
- Check NTP synchronization across all nodes

## Testing

- Verify clock synchronization across all cluster nodes — must be < 1 second
- Check Windows Time Service (w32time) configuration on each node
- Check if the issue correlates with specific nodes (some jobs work, some don't)

## Resolution

- Synchronize clocks across all cluster nodes using NTP (< 1 second tolerance)
- Configure Windows Time Service to use a reliable NTP source
- After syncing, restart IIS on all nodes to clear faulted database state
