# Oracle Fusion MCP Server Documentation

A Model Context Protocol (MCP) server for Oracle Fusion Cloud ERP that provides intelligent metadata exploration, SQL execution, and business process mapping for AI assistants.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Configuration](#configuration)
- [Authentication](#authentication)
- [Licensing](#licensing)
- [Tools Reference](#tools-reference)
- [Resources & Prompts](#resources--prompts)
- [Troubleshooting](#troubleshooting)
- [Security Best Practices](#security-best-practices)

---

## Overview

`ofmcp` is an MCP server that exposes Oracle Fusion metadata tools, SQL linting, and live SQL execution. It enables AI assistants (Claude, Gemini, Codex, Claude Code, etc.) to intelligently query and analyze Oracle Fusion Cloud data.

### Key Features

- **Metadata Exploration** - Browse tables, columns, indexes, and relationships
- **Intelligent Search** - Fuzzy, semantic, and description-based search
- **SQL Validation** - Lint SQL with Oracle-specific fixes and suggestions
- **Live SQL Execution** - Run queries against Oracle Fusion via BI Publisher SOAP
- **Business Process Mapping** - Map processes to tables and understand data flows
- **Multi-Environment Management** - Configure and switch between dev, UAT, prod environments without restarts
- **Authentication Management** - SSO and Basic Auth with token management tools

### Components

| Component | Description |
|-----------|-------------|
| `ofmcp` | MCP server (stdio transport) for MCP clients (Claude Desktop, Gemini, etc.) |
| `metadata.db` | DuckDB cache of Oracle Fusion schema metadata |
| `license.json` | Machine-bound license file |
| `config.json` | Optional configuration file |
| `environments.json` | Optional multi-environment config (managed via tools, not manually) |

---

## Quick Start

1. **Get your machine ID:**
   ```powershell
   ofmcp.exe --print-machine-id
   ```
   *(On macOS: `./ofmcp --print-machine-id`)*

2. **Request a License:**
   - Open: https://license.oraclefusionsql.com/
   - Enter your **email** and **machine ID**.
   - Submit the form.
   - You will receive a verification email. Click the link.
   - You will receive a second email with your license.
   - Save as `license.json` next to the executable.

3. **Configure your MCP Client** (e.g., Claude Desktop, Gemini, Claude Code):
   
   **Windows:**
   ```json
   {
     "mcpServers": {
       "fusion-metadata": {
         "command": "C:\\path\\to\\ofmcp.exe",
         "args": ["--db", "C:\\path\\to\\metadata.db"],
         "env": {
           "FUSION_HOST": "https://your-instance.oraclecloud.com",
           "FUSION_SQL_REPORT_PATH": "/Custom/Financials/RP_ARB.xdo",
           "LICENSE_PATH": "C:\\path\\to\\license.json"
         }
       }
     }
   }
   ```

   **macOS:**
   ```json
   {
     "mcpServers": {
       "fusion-metadata": {
         "command": "/path/to/ofmcp",
         "args": ["--db", "/path/to/metadata.db"],
         "env": {
           "FUSION_HOST": "https://your-instance.oraclecloud.com",
           "FUSION_SQL_REPORT_PATH": "/Custom/Financials/RP_ARB.xdo",
           "LICENSE_PATH": "/path/to/license.json"
         }
       }
     }
   }
   ```

4. **Start using tools:**
   - `get_auth_status` - Check authentication state
   - `authenticate` - Log in via browser SSO
   - `list_tables` - Browse available tables
   - `execute_oracle_sql` - Run live queries

---

## Installation

### Prerequisites

- Chrome/Chromium browser (for SSO authentication)
- Oracle Fusion Cloud instance access
- Valid license file
- **Setup Fusion Reports**: In your Fusion instance, un-archive `DM_ARB.xdm.catalog` and `RP_ARB.xdo.catalog` (found in the `otbireport` folder of this repository: [https://github.com/krokozyab/ofjdbc/tree/master/otbireport](https://github.com/krokozyab/ofjdbc/tree/master/otbireport)) into the `/Shared Folders/Custom/Financials` folder.
  - *Note:* You can use a different folder path, but you will need to update the report path in the connection URL or via `FUSION_SQL_REPORT_PATH` (e.g., `FUSION_SQL_REPORT_PATH=/Custom/folder/RP_ARB.xdo`).

### Invocation

The `ofmcp` executable is designed to be invoked directly by an LLM client (Claude Desktop, Gemini, Claude Code, etc.) as a subprocess. It communicates using the Model Context Protocol (MCP) over stdio.

**Command:**
```bash
./ofmcp --db metadata.db
```

---

## Configuration

### Oracle Fusion Connection

#### FUSION_HOST

**Required**: Yes (unless using [multi-environment management](./multi-environment.md))
**Type**: URL
**Default**: None

Oracle Fusion SaaS instance URL.

**Example:**
```bash
FUSION_HOST=https://you_server.oraclecloud.com
```

**Notes:**
- Must be a valid HTTPS URL
- Do not include trailing slashes
- This is the base URL for your Oracle Fusion Cloud instance
- Can be set via environment variable or `config.json`
- Environment variable takes precedence over config file

**Error if missing:**
```
FUSION_HOST must be set via env or config.json
```

---

#### FUSION_SQL_REPORT_PATH

**Required**: No
**Type**: File Path
**Default**: `/Custom/Financials/RP_ARB.xdo`

BI Publisher report path used for executing SQL queries via SOAP.

**Example:**
```bash
FUSION_SQL_REPORT_PATH=/Custom/folder/RP_ARB.xdo
```

**Notes:**
- This is the path to the BI Publisher report template
- The report must be deployed in your Oracle Fusion instance
- Must have the correct security permissions
- Common paths:
  - `/Custom/Financials/RP_ARB.xdo` (default)
  - `/Custom/folder/RP_ARB.xdo`
  - `/Custom/<module>/RP_ARB.xdo`

---

### Authentication Variables

The system supports two authentication methods:

1. **Bearer Token (OAuth/SSO)** - Default, uses browser-based authentication
2. **Basic Authentication** - Uses username and password

#### FUSION_USER

**Required**: No (Optional for Basic Auth)
**Type**: String
**Default**: None

Username for Basic Authentication.

**Example:**
```bash
FUSION_USER=john.doe@company.com
```

**Notes:**
- Only used if both `FUSION_USER` and `FUSION_PASSWORD` are set
- If set, Basic Authentication is used instead of SSO
- Can be set via environment variable or `config.json`
- Environment variable takes precedence over config file

---

#### FUSION_PASSWORD

**Required**: No (Optional for Basic Auth)
**Type**: String
**Default**: None

Password for Basic Authentication.

**Example:**
```bash
FUSION_PASSWORD=SecurePassword123
```

**Notes:**
- Only used if both `FUSION_USER` and `FUSION_PASSWORD` are set
- If set, Basic Authentication is used instead of SSO
- Can be set via environment variable or `config.json`
- Environment variable takes precedence over config file
- Store securely - consider using config file with restricted permissions

**Authentication Flow:**
```
If FUSION_USER + FUSION_PASSWORD are set:
  → Use Basic Authentication (base64 encoded)
Else:
  → Use Bearer Token Authentication
  → Launch Chrome for SSO
  → Cache tokens for reuse
```

---

### Licensing

#### LICENSE_PATH

**Required**: No
**Type**: File Path
**Default**: `./license.json` (same directory as executable)

Path to the license key file.

**Example:**
```bash
LICENSE_PATH=/Users/username/ofmcp/license.json
```

**Example (Windows):**
```bash
LICENSE_PATH=C:\Users\Administrator\ofmcp\license.json
```

**Notes:**
- If not set, looks for `license.json` next to the executable
- Can be absolute or relative path
- Supports tilde expansion: `~/ofmcp/license.json`
- File must contain valid JSON license data

---

### Browser Configuration

#### FUSION_EXPLORER_CHROME_PATH

**Required**: No
**Type**: File Path
**Default**: Auto-detected

Custom path to Chrome/Chromium executable for SSO authentication.

**Example (macOS):**
```bash
FUSION_EXPLORER_CHROME_PATH=/Applications/Google Chrome.app/Contents/MacOS/Google Chrome
```

**Example (Windows):**
```bash
FUSION_EXPLORER_CHROME_PATH=C:\Program Files\Google\Chrome\Application\chrome.exe
```

**Notes:**
- Only needed if Chrome is not in standard location
- Used for browser-based SSO authentication
- Auto-detection searches these locations:

**macOS:**
- `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`
- `/Applications/Chromium.app/Contents/MacOS/Chromium`

**Windows:**
- `%LOCALAPPDATA%\Google\Chrome\Application\chrome.exe`
- `C:\Program Files\Google\Chrome\Application\chrome.exe`
- `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`

**Chrome Profile Storage:**
- Profile data is saved to `~/.ofed/chrome-profile`
- This maintains SSO session across restarts
- Can be deleted to clear saved sessions

**Error if missing:**
```
Chrome browser not found
```

---

### Configuration Priority

The system resolves Oracle Fusion connection settings in this order:

1. **Per-request `environment` parameter** (highest priority)
2. **Active environment** from `environments.json` (managed via `add_environment` / `switch_environment`)
3. **Environment Variables** set by MCP client
4. **config.json** (next to executable)
5. **Default values** (lowest priority)

> **Note:** If you use `add_environment` to configure environments, those take precedence over env vars. See [Multi-Environment Management](./multi-environment.md) for details.

#### Configuration File Format

Create `config.json` next to the executable:

```json
{
  "FUSION_HOST": "https://your-instance.oraclecloud.com",
  "FUSION_USER": "username",
  "FUSION_PASSWORD": "password",
  "FUSION_SQL_REPORT_PATH": "/Custom/folder/RP_ARB.xdo"
}
```

**Alternative field names** (case-insensitive):
- `FUSION_HOST` or `fusion_host`
- `FUSION_USER` or `fusion_user`
- `FUSION_PASSWORD` or `fusion_password`

---

### Configuration Examples

#### Example 1: SSO Authentication (Recommended)

**Environment Variables:**
```bash
FUSION_HOST=https://you_server.oraclecloud.com
FUSION_SQL_REPORT_PATH=/Custom/folder/RP_ARB.xdo
LICENSE_PATH=/Users/username/ofmcp/license.json
```

**Behavior:**
- Uses browser-based SSO authentication
- Opens Chrome for login
- Caches tokens automatically
- Refreshes tokens when expired

---

#### Example 2: Basic Authentication

**Environment Variables:**
```bash
FUSION_HOST=https://you_server.oraclecloud.com
FUSION_USER=john.doe@company.com
FUSION_PASSWORD=SecurePassword123
FUSION_SQL_REPORT_PATH=/Custom/folder/RP_ARB.xdo
LICENSE_PATH=/Users/username/ofmcp/license.json
```

**Behavior:**
- Uses Basic Authentication
- No browser required
- Credentials sent with every request
- No token caching

---

#### Example 3: MCP Configuration (.mcp.json)

```json
{
  "mcpServers": {
    "fusion-metadata-go": {
      "command": "/path/to/ofmcp",
      "args": ["--db", "/path/to/metadata.db"],
      "env": {
        "FUSION_HOST": "https://you_server.oraclecloud.com",
        "FUSION_SQL_REPORT_PATH": "/Custom/folder/RP_ARB.xdo",
        "LICENSE_PATH": "/path/to/license.json"
      }
    }
  }
}
```

---

#### Example 4: Custom Chrome Path

```bash
FUSION_HOST=https://you_server.oraclecloud.com
FUSION_EXPLORER_CHROME_PATH=/opt/google/chrome/chrome
LICENSE_PATH=/opt/ofmcp/license.json
```

**Use Case:**
- Chrome installed in non-standard location
- Multiple Chrome versions installed
- Using Chromium instead of Chrome
- Docker/container environments

---

## Authentication

The server supports two authentication methods for Oracle Fusion access.

### SSO Authentication (Recommended)

Browser-based Single Sign-On using OAuth tokens.

**Setup:** Do NOT set `FUSION_USER` or `FUSION_PASSWORD`

**Flow:**
1. On first query, Chrome opens to Oracle Fusion login page
2. Complete SSO login in browser
3. Tokens are cached and automatically refreshed

**Benefits:**
- More secure (no stored passwords)
- Tokens expire automatically
- Supports MFA and corporate SSO

### Basic Authentication

Username/password authentication for environments without browser access.

**Setup:** Set both `FUSION_USER` and `FUSION_PASSWORD`

```bash
FUSION_USER=user@company.com
FUSION_PASSWORD=SecurePassword123
```

**Note:** Credentials are sent with every request. Use SSO when possible.

### Authentication Tools

Two MCP tools provide visibility and control over authentication:

#### `get_auth_status`

Check current authentication state.

**Parameters:** None

**Returns:**
- `authenticated` - Whether currently authenticated
- `user` - Authenticated username
- `connection_name` - Oracle Fusion host
- `expires_at` - Token expiry time
- `expires_in` - Human-readable time remaining
- `refresh_needed` - Whether token should be refreshed
- `auth_method` - "sso" or "basic"

**Example Response:**
```
## Authentication Status: Authenticated

- **Connection:** you_server.oraclecloud.com
- **User:** john.doe@company.com
- **Auth Method:** sso
- **Expires In:** 45 minutes
- **Expires At:** 2025-01-31T19:30:00Z
```

#### `authenticate`

Force browser-based re-authentication.

**Parameters:**
- `timeout_seconds` (optional) - Wait time for login (default: 300, min: 30, max: 600)

**Behavior:**
1. Invalidates any cached tokens
2. Launches Chrome browser
3. Waits for user to complete SSO login
4. Caches new tokens
5. Returns success with user info

**Use Cases:**
- Initial authentication
- Re-authenticate after permission changes
- Switch to different user
- Recover from auth errors

---

## Licensing

The server requires a machine-bound license file.

### Getting a License

1. **Generate machine ID:**
   Open a terminal in the folder where `ofmcp` lives and run:
   ```bash
   ./ofmcp --print-machine-id
   ```
   Copy the printed machine ID. You will use it to request a license.

2. **Request a Demo License:**
   - Open: https://license.oraclefusionsql.com/
   - Enter your **email** and **machine ID**.
   - Submit the form.
   - You will receive a verification email. Click the link.
   - You will receive a second email with your license.

3. **Save as `license.json`** next to the executable, or set `LICENSE_PATH`.

### License Format

```json
{
  "payload": {
    "customerId": "acme",
    "machineId": "e620c6fb85e6db527444a4f32e5b2b6568b3694e...",
    "expiresAt": "2025-12-31T23:59:59Z",
    "issuedAt": "2025-01-01T12:00:00Z",
    "plan": "enterprise"
  },
  "sig": "base64-encoded-signature"
}
```

### License Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `licence not found` | Missing file | Place `license.json` next to binary or set `LICENSE_PATH` |
| `licence invalid` | Signature mismatch | Get new license for this machine |
| `licence expired` | Past expiry date | Renew license |

---

## Tools Reference

### Discovery & Metadata

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `list_tables` | List tables with optional schema/module filter | - |
| `list_columns` | List columns, types, and descriptions for a table | `table_name` |
| `table_overview` | Narrative overview with key columns and relationships | `table_name` |
| `index_info` | Index definitions and indexed columns for a table | `table_name` |

### Search

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `search_identifiers` | Weighted fuzzy search for table/column names with Oracle suffix boosts (`_ALL`, `_B`, `_TL`) and noise filtering | `query` |
| `semantic_search` | Business concept search — maps natural language (e.g., "unpaid invoices") to tables via synonyms, module clustering, and relationship density | `query` |
| `search_descriptions` | Multi-token AND search across table/column descriptions and remarks | `query` |

### SQL Tools

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `lint_sql` | Validate SQL against the Oracle schema cache — parses AST, checks table/column existence, suggests fixes | `sql` |
| `suggest_sql` | SQL completion suggestions based on schema context | - |
| `raw_select` | Read-only SELECT on the local metadata cache (DuckDB) — useful for exploring schema without hitting Oracle | `sql` |
| `execute_oracle_sql` | Execute a **live** SQL query against Oracle Fusion via BI Publisher SOAP. Returns results inline (truncated to 50KB). Best for quick lookups | `sql` |
| `execute_oracle_sql_to_file` | Execute a **live** SQL query and save results to a local file (`~/fusion_exports/`). Runs async with automatic pagination. Use for large exports — results are NOT returned inline | `sql` |
| `get_sql_job_status` | Check progress of an async SQL-to-file job (rows fetched, pages completed, elapsed time). Omit `job_id` to list all jobs | - |

### Live REST API

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `rest_call` | Make an HTTP REST call and return the response inline (truncated to 50KB). Works with **any** HTTP endpoint — Oracle Fusion REST APIs with managed SSO auth, or external APIs with bearer/basic/no auth. The agent can call `/describe` on any Fusion resource to discover its schema on the fly, then perform CRUD operations without hardcoded schemas | `url` |
| `rest_call_to_file` | Make a REST call and save the full untruncated response to a local file (`~/fusion_exports/`). Runs async. Set `paginate=true` for Oracle Fusion REST APIs — automatically fetches all pages via `{items, hasMore, next}` pattern and merges into one JSON array | `url` |
| `get_rest_job_status` | Check progress of an async REST-to-file job. Omit `job_id` to list all jobs | - |

### Business Process & Context

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `module_summary` | Summary of an Oracle module (AP, GL, AR, PO, etc.) — key tables, relationships, common queries | `module` |
| `list_business_processes` | List all available business process definitions | - |
| `process_catalog` | Full business process catalog with stages and table mappings | - |
| `business_process_map` | Map a specific process to its primary tables and data flow | `process_code` |
| `relationship_map` | Inbound/outbound FK relationships for a table | `table_name` |
| `integration_flow_mapper` | Data flow dependencies — upstream sources and downstream consumers | `table_name` or `process` |
| `cross_module_analyzer` | Cross-module touchpoints — how a table connects across AP, GL, PO, etc. | `table_name` |
| `scenario_mapper` | Map a business scenario or intent to relevant tables and queries | `scenario` or `intent` |

### Environment Management

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `add_environment` | Add or update a named Oracle Fusion environment (dev, UAT, prod) with SSO or basic auth | `name`, `fusion_host` |
| `list_environments` | List all configured environments and show which one is active | - |
| `switch_environment` | Switch the active environment — all subsequent commands use the new environment | `name` |
| `remove_environment` | Remove a named environment | `name` |

See [Multi-Environment Management](./multi-environment.md) for full details, cross-environment comparison examples, and configuration storage.

### Authentication

| Tool | Description | Required Params |
|------|-------------|-----------------|
| `get_auth_status` | Show current auth state — method, user, token expiry, refresh status. Accepts optional `environment` parameter | - |
| `authenticate` | Force browser-based SSO re-authentication (invalidates cached tokens). Accepts optional `environment` parameter | - |

---

## Resources & Prompts

### MCP Resources

Use these URIs with MCP resource-aware clients:

**Metadata:**
- `ofmcp://metadata/schema` - List tables
- `ofmcp://metadata/table/{table}` - Table columns
- `ofmcp://metadata/indexes/{table}` - Table indexes
- `ofmcp://metadata/relationships/{table}` - Table relationships
- `ofmcp://metadata/module/{module}` - Module summary
- `ofmcp://metadata/process/{process_code}` - Process mapping

**Search:**
- `ofmcp://search/identifiers/{query}` - Identifier search
- `ofmcp://search/semantic/{query}` - Semantic search
- `ofmcp://search/descriptions/{query}` - Description search

### MCP Prompts

Pre-built prompt templates:
- `list_tables`, `list_columns`, `table_overview`
- `search_identifiers`, `semantic_search`
- `lint_sql_and_fix`, `execute_oracle_sql`
- `relationship_map`, `business_process_map`
- `ap_invoice_query` - Domain-specific AP invoice helper

---


## Troubleshooting

### Authentication Issues

| Problem | Solution |
|---------|----------|
| "Access denied" or "Access SOAP" error | User lacks Oracle Fusion privileges. Contact admin. |
| SSO browser doesn't open | Check `FUSION_EXPLORER_CHROME_PATH` or install Chrome |
| Token expired | Run `authenticate` to re-login |
| "Not authenticated" | Run `authenticate` or check `get_auth_status` |

**Clear cached sessions:**
```bash
rm -rf ~/.ofrag/chrome-profile
```

### Configuration Issues

| Problem | Solution |
|---------|----------|
| "FUSION_HOST must be set" | Set environment variable or add to config.json |
| "licence not found" | Place license.json next to binary or set `LICENSE_PATH` |
| "licence invalid" | Machine ID mismatch - get new license |

### Forcing SSO / Overriding System Environment

If you experience issues where a specific user seems to be stuck or the configuration is ignored (e.g., Gemini reports a user from a previous session despite no visible config), you can force SSO mode by explicitly setting empty environment variables in your MCP client configuration.

**Gemini / Claude Config Example:**
```json
"env": {
  "FUSION_HOST": "https://your-instance.oraclecloud.com",
  "FUSION_SQL_REPORT_PATH": "/Custom/Financials/RP_ARB.xdo",
  "FUSION_USER": "",
  "FUSION_PASSWORD": ""
}
```
Setting `FUSION_USER` and `FUSION_PASSWORD` to empty strings will override any system-level environment variables and force the server to use browser-based SSO.

### Connection Issues

| Problem | Solution |
|---------|----------|
| SOAP timeout | Check network connectivity to Oracle Fusion |
| SSL errors | Verify FUSION_HOST uses HTTPS |
| HTTP 500 errors | Check BI Publisher report path and permissions |

---

## Security Best Practices

1. **Protect sensitive files**
   ```bash
   chmod 600 config.json license.json
   ```

2. **Use SSO over Basic Auth** - Tokens expire, passwords don't

3. **Environment-specific configs** - Separate dev/prod settings

4. **Audit access** - Monitor tool usage via logs in `~/.ofrag/logs/`



---

## Support

- **Issues:** https://github.com/krokozyab/of_rag/issues
- **Documentation:** See `docs/` folder for detailed guides
