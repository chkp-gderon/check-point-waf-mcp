# Check Point WAF MCP - Agent Guidance

This file provides workflow guidance for AI agents (Claude, opencode, Cursor, etc.) managing Check Point WAF through the MCP server.

## Environment Setup

Before any operations, ensure environment variables are set correctly:
- `CHECKPOINT_CLIENT_ID`: API client ID from Infinity Portal
- `CHECKPOINT_ACCESS_KEY`: API access key from Infinity Portal
- `CHECKPOINT_REGION`: Must match your tenant's region (`us`, `eu`, `ap`, `au`, `in`)

**IMPORTANT**: Region mismatch causes "Practice does not exist" errors. Verify region before operations.

## Core Concepts

### Practice Visibility
- **Shared**: Accessible by all assets across the tenant
- **Local**: Private to a specific asset/zone (requires `ownerId` when creating)

### Critical Sequencing Rules
1. **Always call `publish_changes`** after create/update/delete operations
2. **Practice must be published before attaching** to an asset
3. **Local practices require `ownerId`** - the asset/zone that will own the practice

## Common Workflows

### Workflow 1: Create New Asset with IPS in Prevent + WebAttacks for Critical

This is the most common request. Follow these exact steps:

**Step 1: Create Shared Practice**
```python
new_web_application_practice(
    name="my-app-practice",
    modes=[{"mode": "Prevent", "subPractice": "IPS"}],
    practice_input={
        "WebAttacks": {"minimumSeverity": "Critical"}
    }
)
```
- Do NOT set `owner_id` (creates shared practice)
- `modes` sets IPS to Prevent mode
- `practice_input` configures WebAttacks to only trigger on Critical severity

**Step 2: Publish the Practice**
```python
publish_changes()
```
- Required! Without this, the practice won't be visible for attachment.

**Step 3: Create the Asset**
```python
new_web_application_asset(
    name="my-web-app",
    upstream_url="https://backend-server:8080",
    urls=["https://myapp.example.com"],
    practice_ids=["practice-id-from-step-1"]
)
```

**Step 4: Publish the Asset Creation**
```python
publish_changes()
```

### Workflow 2: Update Existing Practice (e.g., Change WebAttacks Severity)

```python
update_web_application_practice(
    practice_id="practice-id-here",
    update_input={
        "WebAttacks": {"minimumSeverity": "Critical"}
    }
)
publish_changes()
```

### Workflow 3: Attach Practice to Existing Asset

```python
update_web_application_asset(
    asset_id="asset-id-here",
    update_input={
        "addPractices": [{"practiceId": "practice-id-here"}]
    }
)
publish_changes()
```

### Workflow 4: Remove Practice from Asset

```python
update_web_application_asset(
    asset_id="asset-id-here",
    update_input={
        "removePractices": [{"practiceId": "practice-id-here"}]
    }
)
publish_changes()
```

### Workflow 5: Create Local Practice for Specific Asset

Only use local practices when the practice should be private to one asset:

```python
# Asset must already exist
new_web_application_practice(
    name="local-practice",
    owner_id="asset-id-here",  # Required for local practices
    modes=[{"mode": "Prevent", "subPractice": "IPS"}]
)
publish_changes()
```

## Payload Shape Reference

### update_web_application_asset input
```python
{
    "addPractices": [{"practiceId": "id"}],
    "removePractices": [{"practiceId": "id"}],
    "addProfiles": ["profile-id"],
    "removeProfiles": ["profile-id"],
    "addURLs": ["https://example.com"],
    "removeURLs": ["https://example.com"],
    "name": "new-name",
    "upstreamURL": "https://new-backend:8080",
    "state": "enabled",  # or "disabled"
    "stage": "production"  # or other stage values
}
```

### new_web_application_practice modes
```python
[
    {"mode": "Prevent", "subPractice": "IPS"},
    {"mode": "Detect", "subPractice": "WebAttacks"},
    {"mode": "Disabled", "subPractice": "WebBot"}
]
```
Valid modes: `Prevent`, `Detect`, `Disabled`, `AccordingToPractice`
Valid subPractice: `IPS`, `WebAttacks`, `WebBot`, `Snort`, `FileSecurity`

### practice_input / update_input
```python
{
    "name": "practice-name",
    "WebAttacks": {"minimumSeverity": "Critical"},  # Critical, High, Medium, Low
    "IPS": {
        "severityLevel": "High",  # Critical, High, Medium, Low
        "highConfidence": True,
        "mediumConfidence": True,
        "lowConfidence": False,
        "protectionsFromYear": "2020",
        "performanceImpact": "Medium"  # Low, Medium, High
    },
    "WebBot": {...},
    "Snort": {...}
}
```

## Diagnostic Queries

When things go wrong, use these to understand state:

### Check Current Practices
```python
list_practices(include_private=True)
```
- Shows all practices including local ones
- Check `visibility` field: "Shared" or "Local"

### Check Specific Practice
```python
get_web_application_practice(practice_id="id-here")
```
- Verify practice exists and see its configuration
- Note: May fail if practice is in different region

### Check Asset and Attached Practices
```python
get_web_application_asset(asset_id="id-here")
```
- Shows all practices attached to the asset
- Check `practices` array in response

### Check Region/Authentication
```python
get_overview()
```
- Quick test to verify API connectivity and region

## Common Errors and Solutions

### Error: "Practice with ID ... does not exist"
**Cause**: Region mismatch OR practice not published OR practice is local and `ownerId` not provided

**Solutions**:
1. Verify `CHECKPOINT_REGION` matches your tenant's region
2. Call `publish_changes()` if practice was just created
3. For local practices, include `owner_id` in `update_web_application_practice`

### Error: "Practice with type WebApplication already exist"
**Cause**: Trying to create a local practice when one already exists for that asset

**Solutions**:
1. Update the existing practice instead of creating new
2. Remove existing practice first, then create new one

### Error: "Local Practice creation is not allowed without owner Id"
**Cause**: Trying to create local practice without specifying `owner_id`

**Solution**: Either provide `owner_id` or omit it to create a shared practice

### Error: Bad Request / 400 Error
**Cause**: Incorrect payload shape

**Solutions**:
1. Check payload format matches examples in this guide
2. For `addPractices`/`removePractices`, use `[{"practiceId": "id"}]` not `["id"]`
3. For `modes`, use `[{"mode": "Prevent", "subPractice": "IPS"}]`

## Verification Checklist

After completing any workflow, verify:

- [ ] `publish_changes()` was called after mutations
- [ ] Practice is visible: `list_practices(include_private=True)`
- [ ] Asset has correct practices: `get_web_application_asset(asset_id)`
- [ ] Practice configuration is correct: `get_web_application_practice(practice_id)`

## Quick Reference: Tool Names

| Task | Tool Name |
|------|-----------|
| Create asset | `new_web_application_asset` |
| Update asset | `update_web_application_asset` |
| Delete asset | `delete_asset` |
| Create practice | `new_web_application_practice` |
| Update practice | `update_web_application_practice` |
| List practices | `list_practices` |
| Get practice details | `get_web_application_practice` |
| Get asset details | `get_web_application_asset` |
| Publish changes | `publish_changes` |
| Discard changes | `discard_changes` |
| Raw GraphQL | `raw_graphql_query` |

## Engine-Specific Notes

### Claude / Claude Code
Add this to your `CLAUDE.md` or reference this file:
```
@AGENTS.md
```

### opencode
This guidance is available as a skill: `.opencode/skills/checkpoint-waf.md`

### Cursor
Reference this file in `.cursorrules`:
```
Please read and follow the workflows in AGENTS.md for Check Point WAF operations.
```

### GitHub Copilot
Reference in `.github/copilot-instructions.md`
