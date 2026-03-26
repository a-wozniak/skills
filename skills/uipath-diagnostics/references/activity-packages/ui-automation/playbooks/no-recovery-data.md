---
product: ui-automation
scenario: Healing Agent enabled but no recovery data generated after UI automation failure
dependency: healing-agent
level: product
---

# Healing Agent — No Recovery Data

## Symptoms

- Job faulted with a UI automation exception
- AutopilotForRobots shows Enabled: true, HealingEnabled: true
- But `healing-fixes.json` is empty or doesn't exist
- No files in `healing-agent/uia/` directory

## Triage

- Confirm HA is enabled by checking both `AutopilotForRobots.Enabled` and `AutopilotForRobots.HealingEnabled` (both must be true)
- Do NOT rely on the legacy `EnableAutopilotHealing` field — it can be false even when HA is properly enabled
- Check if the failure activity type is UIAutomation (HA only works with UI activities)
- Download the Healing Agent data for the job using AutopilotForRobotsData tools
- List files first (ListFilesByJobkey), then get read URIs (GetReadDirUriByJobkey)
- Check if the selector was eligible for healing (not all selector types are supported)

## Possible Causes

- The activity uses classic UI automation (`UiPath.UIAutomation.*`) — HA may have limited support for classic activities depending on version
- The failure happened before HA could capture the UI tree (application crashed, window closed)
- The robot machine lost connectivity to Semantic Proxy / LLM Gateway during recovery attempt
- HA analysis timed out — the UI tree was too complex or the LLM response was too slow
- The activity uses image-based targeting which HA doesn't support for recovery

## Resolution

- Verify the activity uses modern UI activities (`UiPath.UIAutomationNext.*`) for full HA support
- Check Semantic Proxy and LLM Gateway health on the deployment
- Check robot machine connectivity to cloud services
- If HA consistently fails to produce data, escalate to platform team with the job ID and robot logs
