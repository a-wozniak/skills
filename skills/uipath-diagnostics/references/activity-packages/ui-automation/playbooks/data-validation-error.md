---
product: ui-automation
scenario: Data validation or business rule exception during automation execution
level: product
---

# Data Validation Error

## Context

Data validation errors occur when the automation processes input data that doesn't meet expected format, type, or business rule constraints. These are often caused by unexpected data from queue items, external systems, or user input — not by the automation code itself.

## Triage

- Get the exception message — it usually identifies the field or value that failed
- If the job processes queue items, get the SpecificContent of failed items
- Check if the error is consistent (all items fail) or data-dependent (some pass, some fail)

## Scenario: queue-item-data-issue

### Symptoms
- Job processes queue items
- Some items succeed, others fail with validation errors
- Error messages reference specific field names or values

### Testing
- Get ALL failed queue items (paginate if >100)
- Get 2-3 successful queue items for comparison
- Compare SpecificContent between failed and successful items
- Identify the specific field/value that triggers the failure
- Check if failures cluster around specific data patterns (null values, special characters, unexpected formats)

### Resolution
- Fix the data at source (dispatcher process or upstream system)
- Add input validation in the performer process before processing
- Handle null/empty values explicitly in the workflow logic

## Scenario: null-reference-in-workflow

### Symptoms
- NullReferenceException in a specific activity
- Occurs when accessing a property or method on a null object
- Often intermittent — depends on runtime data

### Testing
- Locate the faulted activity in source code
- Trace the variable back to its assignment — where does the null come from?
- Check if the variable depends on an upstream activity that can return null (Get Text on missing element, empty queue item field, unset asset)

### Resolution
- Add null checks before accessing the variable
- Ensure upstream activities handle "not found" cases explicitly
- Use default values where appropriate
