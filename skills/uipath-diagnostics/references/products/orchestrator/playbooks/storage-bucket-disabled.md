---
product: orchestrator
scenario: FileSystem provider not available when creating or configuring storage buckets
level: product
---

# Storage Bucket FileSystem Provider Disabled

## Symptoms

- FileSystem provider not available in the storage bucket provider dropdown
- Only Azure, S3, MinIO, and Orchestrator native providers visible

## Triage

- Check Orchestrator configuration for storage bucket providers
- Confirm the user is looking for the FileSystem provider specifically

## Testing

- Check the `Buckets.AvailableProviders` configuration setting
- FileSystem is disabled by default and must be explicitly enabled

## Resolution

- Enable the FileSystem provider in Orchestrator configuration by adding it to `Buckets.AvailableProviders`
- Restart Orchestrator after configuration change
- Note: FileSystem provider stores files on the Orchestrator server's local disk — not suitable for clustered deployments unless using a shared filesystem
