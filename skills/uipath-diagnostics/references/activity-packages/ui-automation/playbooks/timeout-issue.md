---
product: ui-automation
scenario: UI automation activity exceeded its timeout waiting for an element or application state
level: product
---

# Timeout Issue

## Context

UI automation activities wait for elements to appear before interacting. The default timeout is typically 30 seconds. A timeout means the element either never appeared or the application didn't reach the expected state within the wait period.

TimeoutException is ambiguous — it could be a UI timeout (Check App State, element wait) or a non-UI timeout (HTTP request, queue transaction). The triage step must confirm it's UI-related before following this playbook.

## Triage

- Check the faulted activity type in trace data — confirm it's a UI automation activity
- If the faulted activity is NOT a UI automation type (e.g., HTTP Request, Send Mail), this playbook doesn't apply
- Get the configured timeout value from the activity properties
- Get the actual duration from the trace span
- Check if the target element exists but is not visible/interactable
- Check robot session info — is the machine locked or in a disconnected RDP session?
- Check if the automation uses Picture-in-Picture (PiP) mode — PiP has different element visibility rules

## Scenario: element-never-appeared

### Symptoms
- Activity waited the full timeout duration
- The target element genuinely doesn't exist (page didn't load, navigation didn't complete, wrong page)

### Testing
- Compare the expected page/state with what the application was actually showing
- Check if a previous navigation or login step failed silently
- Check if the application is slower than usual (server-side delay)

### Resolution
- If the application is genuinely slower: increase the timeout value in the activity properties
- If the wrong page is displayed: fix the upstream navigation logic
- Add a Check App State activity before the failing activity with appropriate wait conditions

## Scenario: timing-gap

### Symptoms
- Activity threw TimeoutException but the element exists (works on manual retry)
- Application loads slowly or has dynamic content that appears after initial render

### Testing
- Check if the activity duration is close to the configured timeout (within 1-2 seconds = genuine timeout)
- Check if other activities in the same workflow also show slow durations
- Check if the issue is intermittent (works sometimes, fails sometimes)

### Resolution
- Add explicit wait (Check App State or Element Exists) before the activity
- Increase the timeout if the application legitimately needs more time
- Do NOT just increase timeout blindly — find out why the element is delayed
