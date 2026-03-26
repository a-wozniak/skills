---
product: orchestrator
scenario: Asset not found or permission denied when automation reads an asset at runtime
level: product
---

# Asset Not Found

## Symptoms

- Job fails with "GetAsset" exception
- Error: "Asset not found" or "does not have the required permissions to access the asset"
- Process works in one folder but fails in another

## Triage

- Get the exact error message from the job traces
- Identify which asset name the process is trying to read
- Identify which folder the job is running in

## Hypothesis Generation

- Asset does not exist in the target folder
- Asset exists but with a different name (case-sensitive mismatch)
- Asset exists in a parent folder but inheritance is not enabled
- Robot/user does not have permission to read assets in the folder

## Testing

- List assets in the job's folder: `uip or assets list {folderId} --format json`
- Compare asset name in source code vs Orchestrator (exact match, case-sensitive)
- Check folder hierarchy — does the asset exist in a parent folder?
- Check robot account permissions in the folder

## Resolution

- Create the missing asset in the correct folder
- Fix the asset name to match source code exactly
- Enable folder inheritance if the asset should be shared from a parent folder
- Grant the robot account permission to access assets in the folder
