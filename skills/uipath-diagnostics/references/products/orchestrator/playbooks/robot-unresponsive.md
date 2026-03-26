---
product: orchestrator
scenario: Robot marked as Unresponsive with heartbeat loss
level: product
---

# Robot Unresponsive

## Symptoms

- Robot status shows "Unresponsive" after ~2 minutes
- Commands delayed by up to 30 seconds
- Jobs cannot be dispatched to the robot

## Triage

- Check robot session status via Sessions_Get
- Check if the issue affects one robot or multiple

## Hypothesis Generation

- Robot service not running on the host machine
- Network interruption between robot and Orchestrator
- SignalR transport misconfiguration
- Host machine rebooted or went to sleep

## Testing

- Verify the UiPath Robot Service is running on the host
- Check network connectivity from the robot host to the Orchestrator URL
- Check if other robots on the same machine/network are also unresponsive
- Review Windows Event Log on the robot host for service crashes or restarts

## Resolution

- Restart the UiPath Robot Service on the host machine
- Fix network connectivity issues (firewall, proxy, DNS)
- For SignalR issues: verify WebSocket support is enabled on the network path
