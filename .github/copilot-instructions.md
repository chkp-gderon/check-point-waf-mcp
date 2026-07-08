# Check Point WAF MCP - Copilot Instructions

Use the Check Point WAF MCP server tools to manage assets, practices, and policy state.

## Required Environment Variables

- `CHECKPOINT_CLIENT_ID`
- `CHECKPOINT_ACCESS_KEY`
- `CHECKPOINT_REGION` (`us`, `eu`, `ap`, `au`, `in`)

## Operational Rules

- Call `publish_changes` after create, update, or delete operations.
- Publish a practice before attaching it to an asset.
- Use `owner_id` only for local practices (asset-scoped/private).
- For asset discovery and selection, use only user-defined assets (`list_assets(user_defined=true)`).
- Exclude the built-in `Any Service` asset from WAF asset lists and workflows.

## Common Workflows

### Create asset with IPS Prevent and WebAttacks Critical

1. Create shared practice with IPS mode `Prevent` and `WebAttacks.minimumSeverity=Critical`.
2. Call `publish_changes`.
3. Create the web application asset and attach the practice.
4. Call `publish_changes`.

### Update existing practice severity

1. Call `update_web_application_practice`.
2. Call `publish_changes`.

### Attach or remove practice on existing asset

1. Call `update_web_application_asset` with `addPractices` or `removePractices`.
2. Call `publish_changes`.

## Troubleshooting

- "Practice does not exist" often means region mismatch or unpublished changes.
- Verify connectivity with `get_overview`.
- Inspect current objects with `list_practices(include_private=True)` and `get_web_application_asset`.

## Behavior Removal Workflow (Important)

- Prefer v1 for behavior deletion: `deleteBehavior(id: ID!)` (returns Boolean). v2 may not expose this mutation.
- If `deleteBehavior` fails with `object-in-use`, detach the behavior first via v1:
	`updatePracticeBehaviors(practiceId, containerId, removeBehaviors:[behaviorId])`.
- After behavior create/update/delete operations, always call `publish_changes`.
- Generic example: remove a behavior by first detaching it from the relevant practice/container,
	then deleting it, with `containerId` set to the owning asset/zone ID.

## Exception Query Workflow (Avoid Common Errors)

- Preferred tool: `get_asset_exceptions_logic(asset_id)` for behavior mapping plus exception `match`/`actions` in one call.
- Use `get_asset_exceptions(asset_id)` only when metadata (`objectStatus`, `time`, `by`) is specifically needed.
- `get_exception_parameter(exception_parameter_id)` expects an ExceptionParameter/behavior ID, not an individual exception ID.
- Do not use `raw_graphql_query` for exception retrieval unless explicitly requested; schema shape mismatches commonly cause 400 errors.

## Exception Response Format

When the user asks for exceptions on a WAF asset:

- Show the behavior-to-asset mapping for the requested asset.
- For each exception, show only:
	- Exception ID
	- Match logic (`match`)
	- Action logic (`actions`)
- Do not include timestamps/creation time unless explicitly requested.
- Do not include a "Behaviors with exceptions vs none" summary block.
