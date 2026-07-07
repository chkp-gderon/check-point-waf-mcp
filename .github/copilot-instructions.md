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
