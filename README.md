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
- **list_profiles** - List all WAF security profiles
- **get_profile** - Get details of a specific security profile
- **list_agents** - List all connected agents/gateways
- **list_practices** - List all security practices
- **get_web_application_practice** - Get detailed Web Application Practice (WAF) configuration
- **get_exception_parameter** - Get detailed exception parameter configuration including exceptions, actions, and supported practice types
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

### Utility
- **raw_graphql_query** - Execute arbitrary GraphQL queries/mutations (use for schema introspection or unsupported operations)

## Prerequisites

- **Python 3.11+**
- **Check Point Infinity Portal API key** - A client ID and access key pair from the [Infinity Portal](https://portal.checkpoint.com/)

## Installation

### Using uv (recommended)

```bash
uvx checkpoint-waf-mcp
```

### Using pip

```bash
pip install checkpoint-waf-mcp
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
        "CHECKPOINT_CLIENT_ID": "your-client-id",
        "CHECKPOINT_ACCESS_KEY": "your-access-key",
        "CHECKPOINT_REGION": "us"
      }
    }
  }
}
```

### Example Prompts

Once connected, you can ask things like:

- "List all my WAF assets"
- "Show tuning suggestions for asset X"
- "Create a new web application asset for my API"
- "Publish my pending changes"
- "Show me the exceptions for asset X"
- "Show me the details for both exceptions"

## API Reference

For more information about the Check Point WAF API, see the [Management API Reference](https://waf-doc.inext.checkpoint.com/references/management-api).
