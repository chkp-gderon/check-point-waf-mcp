# Check Point WAF MCP Server

<!-- Badges placeholder -->
[![Python](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)

An [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) server that provides tools for managing Check Point WAF through its GraphQL API.

## Features

### Queries
- **list_assets** - List all WAF assets with optional filtering
- **get_asset** - Get detailed information about a specific asset
- **get_web_application_asset** - Get full details of a Web Application asset including URLs and proxy settings
- **get_asset_tuning** - Get tuning suggestions for an asset
- **get_asset_tuning_review** - Get tuning review decisions for an asset
- **get_asset_statistics** - Get traffic and security statistics for an asset
- **get_asset_exceptions** - Get exceptions for an asset including metadata (objectStatus, timestamp, created by)
- **get_asset_exceptions_logic** - Get behavior-to-asset mapping and each exception's `match` + `actions` in one call
- **list_profiles** - List all WAF security profiles
- **get_profile** - Get details of a specific security profile
- **list_agents** - List all connected agents/gateways
- **list_practices** - List all security practices
- **get_web_application_practice** - Get detailed Web Application Practice (WAF) configuration
- **get_exception_parameter** - Get detailed exception parameter configuration (expects ExceptionParameter/behavior ID, not exception ID)
- **get_behavior** - Get behavior details by ID, including behavior type, visibility, and usage count
- **get_overview** - Get a high-level overview of configured objects
- **list_log_triggers** - List all log trigger configurations

### Mutations
- **publish_changes** - Publish all pending changes
- **discard_changes** - Discard all pending changes
- **enforce_policy** - Enforce the latest published policy on all connected agents
- **new_web_application_asset** - Create a new Web Application asset
- **update_web_application_asset** - Update an existing Web Application asset
- **upload_client_mtls_trusted_ca_chain** - Upload a trusted CA chain file for protected client mTLS configuration
- **upload_server_mtls_trusted_ca_chain** - Upload a trusted CA chain file for server-side mTLS configuration
- **delete_asset** - Delete an asset
- **new_web_application_practice** - Create a new Web Application (WAF) practice
- **update_web_application_practice** - Update a Web Application (WAF) practice
- **update_practice_behaviors** - Add or remove behavior assignments for a practice/container pair

### Utility
- **raw_graphql_query** - Execute arbitrary GraphQL queries/mutations (use for schema introspection or unsupported operations)

## Prerequisites

- **Python 3.11+**
- **Check Point Infinity Portal API key** - A client ID and access key pair from the [Infinity Portal](https://portal.checkpoint.com/)

## Installation

### Using uv (recommended)

```bash
uvx --from . checkpoint-waf-mcp
```

### Using pip

```bash
pip install -e .
```

## Configuration

The server is configured via environment variables:

| Variable | Description | Required |
|---|---|---|
| `CHECKPOINT_CLIENT_ID` | API client ID from Infinity Portal | Yes |
| `CHECKPOINT_ACCESS_KEY` | API access key from Infinity Portal | Yes |
| `CHECKPOINT_REGION` | Data center region | No (default: `us`) |
| `CHECKPOINT_WAF_MCP_VERBOSE` | Enable server startup logs to stderr (`1` enabled, `0` disabled) | No (default: `1`) |

### Available Regions

| Region Code | Location |
|---|---|
| `us` | United States |
| `eu` | Europe |
| `ap` | Asia Pacific |
| `au` | Australia |
| `in` | India |


## Usage

### Run Locally

Start the MCP server directly from this repository:

```bash
python -m checkpoint_waf_mcp.server
```

### Claude Desktop

Add the following to your Claude Desktop configuration file (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "checkpoint-waf": {
      "command": "uvx",
      "args": ["checkpoint-waf-mcp"],
      "env": {
        "CHECKPOINT_CLIENT_ID": "${env:CHECKPOINT_CLIENT_ID}",
        "CHECKPOINT_ACCESS_KEY": "${env:CHECKPOINT_ACCESS_KEY}",
        "CHECKPOINT_REGION": "${env:CHECKPOINT_REGION}"
      }
    }
  }
}
```

### GitHub Copilot (VS Code)

Add the following to your VS Code MCP configuration file (`mcp.json`):

```json
{
  "servers": {
    "checkpoint-waf": {
      "type": "stdio",
      "command": "checkpoint-waf-mcp",
      "args": [],
      "env": {
        "CHECKPOINT_CLIENT_ID": "${env:CHECKPOINT_CLIENT_ID}",
        "CHECKPOINT_ACCESS_KEY": "${env:CHECKPOINT_ACCESS_KEY}",
        "CHECKPOINT_REGION": "${env:CHECKPOINT_REGION}"
      }
    }
  }
}
```

Alternatively, use the Command Palette to install this MCP server in VS Code:

1. Open the Command Palette and run `MCP: Add Server`.
2. Choose `Stdio server`.
3. Use this command if installed from this repository:

```text
checkpoint-waf-mcp
```

After adding the server, open Copilot Chat in Agent mode and run prompts such as "list all my WAF assets".

### Example Prompts

Once connected, you can ask things like:

- "List all my WAF assets"
- "Show tuning suggestions for asset X"
- "Create a new web application asset for my API"
- "Publish my pending changes"
- "Show me the exceptions for asset X"
- "Show me exception logic for asset X"
- "Show me the details for both exceptions"
- "Show me behavior details for behavior ID X"
- "Remove behavior Y from asset Z"

## API Reference

For more information about the Check Point WAF API, see the [Management API Reference](https://waf-doc.inext.checkpoint.com/references/management-api).
