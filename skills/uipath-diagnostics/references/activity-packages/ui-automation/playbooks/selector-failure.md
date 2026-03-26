---
product: ui-automation
scenario: UI element not found — SelectorNotFoundException, UiElementNotFoundException, ElementNotInteractableException, or NodeNotFoundException during activity execution
level: product
requirements:
  - id: source_code_path
    scope: [process, activity]
    prompt: Ask for the project source code — selectors need source code analysis, cannot diagnose without it
    auto_resolve: null
    required: true
    deferrable: false
  - id: target_application
    scope: [activity]
    prompt: Ask which application the automation targets (e.g., SAP, browser, desktop app)
    auto_resolve: null
    required: false
---

# Selector Failure

## Context

When a UI automation activity executes, it uses an XML selector to locate the target element in the live UI tree. A selector failure means the selector didn't match any element, or the element was found but can't be interacted with.

Common causes:
- Target application UI changed (redesign, update, dynamic content)
- Element attribute became dynamic (index shifted, name changed per session)
- Element hidden behind an overlay, popup, or dialog
- Timing issue — element not loaded yet when activity executed
- Wrong application window targeted

## Triage

- Get the faulted activity name and selector from job traces
- Check if Healing Agent was enabled on the process (AutopilotForRobots field in job info)
- If HA was enabled, check for `healing-fixes.json` — HA may have already identified the fix
- If HA was disabled, note this as a configuration finding (enable HA for future diagnostics)
- Download the video recording for the job if available (VideoRecording_ListReadUrisByJobkey)
- Check if the target application changed (version update, UI redesign)

## Scenario: healing-agent-fix-available

When `healing-fixes.json` exists and contains a matching entry for the faulted activity.

### Symptoms
- Job faulted with selector exception
- Healing Agent was enabled
- `healing-fixes.json` exists with an entry matching the faulted activity's `ActivityRefId`

### Testing
- Match entry by `ActivityRefId` (preferred) or `activityName` + `workflowFile` (fallback)
- Extract the `enhancedTarget` (for `update-target` fixes) or `clickTarget` (for `dismiss-popup` fixes)
- Compare failed selector vs recommended selector
- Check confidence score and strategy name

### Resolution
- For `update-target`: update the activity's selector in XAML with the HA-recommended selector
- For `dismiss-popup`: add a Click activity before the failing activity to dismiss the popup
- Match XAML activity by `ActivityRefId` → `IdRef` attribute (unique within workflow)
- Apply proper XML encoding when editing selectors: encode `&` first, then `<`, `>`, `'`, `"`

## Scenario: healing-agent-disabled

When the job failed with a UI automation exception but Healing Agent was not enabled.

### Symptoms
- Job faulted with selector exception
- `AutopilotForRobots` is null or disabled in job info
- No `healing-fixes.json` available

### Testing
- Confirm UIAutomation failure via trace spans (activity types: `UiPath.UIAutomationNext.*`, `UiPath.UIAutomation.*`, `UiPath.Core.Activities.Click`, etc.)
- If trace unavailable, infer from exception type (SelectorNotFoundException is definitively UI)
- TimeoutException is ambiguous — only classify as UI if trace confirms UI activity type

### Resolution
- Enable Healing Agent on the process: update release ProcessSettings with `{"AutopilotForRobots":{"Enabled":true,"HealingEnabled":true}}`
- Optionally restart the job — if it fails again, HA will capture full diagnostic data for a more detailed analysis
- Root cause still needs investigation via source code analysis or manual selector comparison

## Scenario: no-healing-data-manual-investigation

When HA was enabled but didn't produce fixes (HA couldn't find an alternative), or source code is available for manual analysis.

### Symptoms
- Selector exception thrown
- No matching entry in `healing-fixes.json`, or file doesn't exist despite HA being enabled
- Source code available

### Testing
- Locate the faulted activity in XAML by `IdRef`
- Extract the selector from the XAML (decode XML encoding: `&amp;` → `&`, `&lt;` → `<`, etc.)
- Analyze the selector: which attributes are used? Are any dynamic (idx, tableRow, etc.)?
- Check selector attributes — fragile selectors use title/name, robust selectors use automationId/className
- Check if the target application has changed recently
- Compare against Object Repository if available
- If HA data exists but no fix was produced: check if the selector was eligible for healing (not all selector types are supported), check the confidence threshold — healing may have been attempted but rejected due to low confidence, compare the original selector with what HA proposed as alternative

### Resolution
- Update the selector to use more stable attributes (aaname, automationid, role) instead of volatile ones (idx, tableCol, tableRow)
- Add wildcard matching for dynamic portions: `name='Invoice*'` instead of `name='Invoice_20250319'`
- Consider adding a Check App State activity before the failing activity to wait for the element

## Shortcuts

### NodeNotFoundException
- **Match**: error message contains "NodeNotFoundException"
- **Root cause**: the application might have changed, the selector might not be correct on the activity, or a popup might have blocked the application
- **Fix**: if Healing Agent is disabled, enable it and run the job again. Otherwise check that the application version and selector are correct.
- **Still test**: yes — check if healing agent is enabled on the job and/or process
