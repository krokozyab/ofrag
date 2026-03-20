# Multi-Environment Management

Configure and switch between multiple Oracle Fusion environments (dev, UAT, prod) directly from the AI agent conversation — no config file edits or restarts required.

---

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Tools Reference](#tools-reference)
  - [add_environment](#add_environment)
  - [list_environments](#list_environments)
  - [switch_environment](#switch_environment)
  - [remove_environment](#remove_environment)
- [Authentication Types](#authentication-types)
  - [SSO (Browser-Based)](#sso-browser-based)
  - [Basic Authentication](#basic-authentication)
- [Using Environments with Queries](#using-environments-with-queries)
  - [Active Environment](#active-environment)
  - [Per-Request Environment Override](#per-request-environment-override)
- [Configuration Storage](#configuration-storage)
  - [File Format](#file-format)
  - [File Location](#file-location)
  - [Security](#security)
- [Configuration Priority](#configuration-priority)
- [Migration from Legacy Configuration](#migration-from-legacy-configuration)
- [Examples](#examples)
  - [Typical Multi-Environment Setup](#typical-multi-environment-setup)
  - [Cross-Environment Data Comparison](#cross-environment-data-comparison)
  - [Mixed Authentication Methods](#mixed-authentication-methods)
  - [MCP Client Configuration](#mcp-client-configuration)
- [Troubleshooting](#troubleshooting)

---

## Overview

In many Oracle Fusion implementations, teams work across multiple environments:

| Environment | Purpose |
|-------------|---------|
| **DEV** | Development and initial testing |
| **TEST / SIT** | System integration testing |
| **UAT** | User acceptance testing |
| **PROD** | Production |

Previously, switching between environments required editing MCP client configuration and restarting the server. The multi-environment feature lets you:

- **Add environments interactively** — provide URL, auth type, and credentials directly in the agent conversation
- **Switch instantly** — change the active environment with one command, no restart needed
- **Override per-request** — target a specific environment for a single query without switching
- **Mix auth methods** — use SSO for dev, basic auth for UAT, SSO for prod — each environment has its own settings

---

## Quick Start

**Step 1:** Add your environments.

```
→ add_environment(name="dev", fusion_host="https://acme-dev.fa.us2.oraclecloud.com", auth_type="sso")
→ add_environment(name="uat", fusion_host="https://acme-uat.fa.us2.oraclecloud.com", auth_type="basic", username="test_user@company.com", password="SecurePass123")
→ add_environment(name="prod", fusion_host="https://acme-prod.fa.us2.oraclecloud.com", auth_type="sso")
```

**Step 2:** Check what's configured.

```
→ list_environments
```

Output:
```
## Configured Environments

**Active:** dev

| Name | Host | Auth | User |
|------|------|------|------|
| dev ✅ | https://acme-dev.fa.us2.oraclecloud.com | sso | (SSO) |
| prod | https://acme-prod.fa.us2.oraclecloud.com | sso | (SSO) |
| uat | https://acme-uat.fa.us2.oraclecloud.com | basic | test_user@company.com |
```

**Step 3:** Switch and query.

```
→ switch_environment(name="uat")
→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL")
```

---

## Tools Reference

### add_environment

Add a new environment or update an existing one.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Short name for the environment (e.g., `dev`, `uat`, `prod`, `sandbox`) |
| `fusion_host` | string | Yes | Oracle Fusion host URL (e.g., `https://acme-dev.fa.us2.oraclecloud.com`) |
| `auth_type` | string | No | `sso` (default) or `basic` |
| `username` | string | No | Username for basic auth. Required if `auth_type=basic` |
| `password` | string | No | Password for basic auth. Required if `auth_type=basic` |
| `report_path` | string | No | Custom BI Publisher report path (e.g., `/Custom/CloudSQL/RP_ARB.xdo`). Uses server default if omitted |

**Behavior:**
- If `name` already exists, the environment is **updated** (all fields are overwritten)
- The **first** environment added becomes active automatically
- If `auth_type` is `basic`, both `username` and `password` are required
- If `auth_type` is `sso` (or omitted), `username`/`password` are ignored — browser-based SSO is used

**Examples:**

```
# SSO environment (most common)
→ add_environment(name="dev", fusion_host="https://acme-dev.fa.us2.oraclecloud.com")

# Basic auth environment
→ add_environment(
    name="uat",
    fusion_host="https://acme-uat.fa.us2.oraclecloud.com",
    auth_type="basic",
    username="john.doe@company.com",
    password="SecurePass123"
  )

# With custom report path
→ add_environment(
    name="prod",
    fusion_host="https://acme-prod.fa.us2.oraclecloud.com",
    auth_type="sso",
    report_path="/Custom/CloudSQL/RP_ARB.xdo"
  )

# Update existing environment
→ add_environment(name="dev", fusion_host="https://acme-dev2.fa.us2.oraclecloud.com")
```

---

### list_environments

List all configured environments and show which one is active.

**Parameters:** None.

**Output:**

Shows a table with all environments, their hosts, auth methods, and users. The active environment is marked with a checkmark.

```
## Configured Environments

**Active:** dev

| Name | Host | Auth | User |
|------|------|------|------|
| dev ✅ | https://acme-dev.fa.us2.oraclecloud.com | sso | (SSO) |
| prod | https://acme-prod.fa.us2.oraclecloud.com | sso | (SSO) |
| uat | https://acme-uat.fa.us2.oraclecloud.com | basic | john.doe@company.com |
```

If no environments are configured, shows a message indicating legacy configuration is in use.

---

### switch_environment

Switch the active environment. All subsequent commands will use the new environment.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Name of the environment to activate |

**Behavior:**
- Immediately changes the active environment
- All subsequent tool calls (`execute_oracle_sql`, `rest_call`, `authenticate`, etc.) use the new environment
- Does **not** invalidate existing SSO tokens — if you switch back, cached tokens are still valid
- Persisted to `environments.json` — survives server restarts

**Example:**
```
→ switch_environment(name="prod")

## Switched to Environment: prod

- **Host:** https://acme-prod.fa.us2.oraclecloud.com
- **Auth Type:** sso

All subsequent commands will use this environment.
```

**Error if not found:**
```
environment "staging" not found. Available: dev, uat, prod
```

---

### remove_environment

Remove a named environment.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Name of the environment to remove |

**Behavior:**
- Permanently deletes the environment from `environments.json`
- If the **active** environment is removed:
  - Another remaining environment becomes active (non-deterministic which one)
  - If no environments remain, the system falls back to env vars from MCP client config
- Cached SSO tokens for the removed environment remain in memory until server restart (they are harmless)

**Example:**
```
→ remove_environment(name="sandbox")

## Removed Environment: sandbox

Active environment is now: **dev**
```

---

## Authentication Types

Each environment can use a different authentication method. The auth type is set when adding the environment and determines how credentials are handled for all requests to that environment.

### SSO (Browser-Based)

**Set with:** `auth_type="sso"` (default)

Browser-based Single Sign-On using Oracle's identity provider. This is the recommended method for interactive use.

**How it works:**
1. On first request (or when token expires), Chrome opens automatically
2. User completes SSO login in the browser
3. The system captures the authentication token
4. Token is cached in memory and reused for subsequent requests
5. Token is automatically refreshed when nearing expiry

**Advantages:**
- No passwords stored on disk
- Works with MFA / two-factor authentication
- Tokens expire automatically
- Follows organizational SSO policies

**Requirements:**
- Chrome or Chromium browser installed
- User can access the Oracle Fusion login page
- Set `FUSION_EXPLORER_CHROME_PATH` if Chrome is in a non-standard location

**Workflow:**
```
→ add_environment(name="dev", fusion_host="https://acme-dev.fa.us2.oraclecloud.com")
→ authenticate(environment="dev")            # Opens Chrome for login
→ execute_oracle_sql(sql="SELECT ...")        # Uses cached token
```

### Basic Authentication

**Set with:** `auth_type="basic"`, plus `username` and `password`

Username/password authentication sent with every request as a Base64-encoded header.

**How it works:**
1. Credentials are stored in `environments.json` (file permissions: 0600)
2. Every request includes an `Authorization: Basic <base64>` header
3. No browser interaction needed
4. No token caching or refresh — credentials are sent each time

**Advantages:**
- No browser required (works in headless/server environments)
- No interactive login step
- Simpler setup for automation / CI pipelines

**Considerations:**
- Password is stored on disk in `environments.json`
- Ensure the file has restricted permissions (`chmod 600`)
- Does not work with MFA-enforced accounts
- Consider using SSO for production environments

**Workflow:**
```
→ add_environment(
    name="uat",
    fusion_host="https://acme-uat.fa.us2.oraclecloud.com",
    auth_type="basic",
    username="test_user@company.com",
    password="SecurePass123"
  )
→ execute_oracle_sql(sql="SELECT ...")        # Credentials sent automatically
```

---

## Using Environments with Queries

### Active Environment

After configuring environments, the **active** environment is used by default for all tool calls:

```
→ switch_environment(name="dev")
→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL")
  # ↑ Runs against dev

→ switch_environment(name="prod")
→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL")
  # ↑ Runs against prod
```

The following tools respect the active environment:

| Tool | What uses the environment |
|------|--------------------------|
| `execute_oracle_sql` | SOAP endpoint, authentication |
| `execute_oracle_sql_to_file` | SOAP endpoint, authentication, report path |
| `rest_call` | URL resolution (relative paths), managed auth |
| `rest_call_to_file` | URL resolution, managed auth |
| `authenticate` | Which host to open in browser |
| `get_auth_status` | Which host's auth state to check |

### Per-Request Environment Override

Every tool listed above accepts an optional `environment` parameter that overrides the active environment **for that single request only**:

```
→ switch_environment(name="dev")               # dev is active

→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL")
  # ↑ Runs against dev (active)

→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL", environment="prod")
  # ↑ Runs against prod (override), dev remains active

→ execute_oracle_sql(sql="SELECT COUNT(*) FROM AP_INVOICES_ALL")
  # ↑ Still runs against dev (active unchanged)
```

This is useful for:
- **Comparing data** across environments without switching back and forth
- **Quick checks** on a non-active environment
- **Automation scripts** that target specific environments

---

## Configuration Storage

### File Format

Environments are stored in `environments.json`:

```json
{
  "active_environment": "dev",
  "environments": {
    "dev": {
      "name": "dev",
      "fusion_host": "https://acme-dev.fa.us2.oraclecloud.com",
      "auth_type": "sso",
      "report_path": "/Custom/CloudSQL/RP_ARB.xdo"
    },
    "uat": {
      "name": "uat",
      "fusion_host": "https://acme-uat.fa.us2.oraclecloud.com",
      "auth_type": "basic",
      "fusion_user": "test_user@company.com",
      "fusion_password": "SecurePass123"
    },
    "prod": {
      "name": "prod",
      "fusion_host": "https://acme-prod.fa.us2.oraclecloud.com",
      "auth_type": "sso"
    }
  }
}
```

**Field reference:**

| Field | Type | Description |
|-------|------|-------------|
| `active_environment` | string | Name of the currently active environment |
| `environments` | object | Map of environment name to configuration |
| `environments[].name` | string | Environment name (matches the key) |
| `environments[].fusion_host` | string | Oracle Fusion instance URL |
| `environments[].auth_type` | string | `sso` or `basic` |
| `environments[].fusion_user` | string | Username (basic auth only) |
| `environments[].fusion_password` | string | Password (basic auth only) |
| `environments[].report_path` | string | Custom BI Publisher report path |

### File Location

`environments.json` is stored in the **same directory as the executable**:

```
/path/to/
├── ofmcp                ← binary
├── metadata.db          ← schema cache
├── environments.json    ← multi-environment config (managed via tools)
├── embeddings.db        ← optional vector search
└── rest_catalog.db      ← optional REST catalog
```

### Security

The file is created with `0600` permissions (owner read/write only). Additional best practices:

1. **Never commit `environments.json` to version control** — add it to `.gitignore`
2. **Restrict file permissions:**
   ```bash
   chmod 600 environments.json
   ```
3. **Prefer SSO** for environments where security is critical (prod)
4. **Use basic auth** only where necessary (automation, CI, headless environments)
5. **Rotate credentials** regularly for basic auth environments
6. **Audit access** — the file stores passwords in plaintext for basic auth environments

---

## Configuration Priority

When resolving which Oracle Fusion host and credentials to use, the system follows this priority order:

```
1. Per-request "environment" parameter       (highest priority)
       ↓
2. Active environment from environments.json
       ↓
3. Environment variables from MCP client     (lowest priority)
```

**Detailed resolution flow:**

```
Tool called with environment="prod"?
  → YES: Use the "prod" environment from environments.json
  → NO:  Is there an active environment in environments.json?
           → YES: Use the active environment
           → NO:  Is FUSION_HOST env var set (from MCP client config)?
                    → YES: Use env var (+ FUSION_USER/FUSION_PASSWORD if set)
                    → NO:  Error: FUSION_HOST not configured
```

This means:
- **Existing setups continue to work** — if you don't use `add_environment`, env vars from your MCP client config are used as before
- **Environments take precedence** — once you add environments, they override env vars
- **Per-request overrides win** — specifying `environment="X"` always uses that environment

---

## Migration from Legacy Configuration

If you're currently using env vars in your MCP client config, you can migrate to the multi-environment system without breaking anything.

### Option 1: Gradual Migration (Recommended)

Your existing env vars continue to work. Add environments alongside them:

```
# Your current env vars (FUSION_HOST etc.) still work
# Now add environments for additional instances:
→ add_environment(name="uat", fusion_host="https://uat.fa.us2.oraclecloud.com", auth_type="basic", username="user", password="pass")
```

When no environment is active and no `environment` parameter is provided, tools fall back to env vars from your MCP client config.

### Option 2: Full Migration

Convert your existing setup to an environment:

```
# If your MCP client config env section has:
#   FUSION_HOST=https://dev.fa.us2.oraclecloud.com
#   FUSION_USER=user@company.com
#   FUSION_PASSWORD=pass123

→ add_environment(
    name="dev",
    fusion_host="https://dev.fa.us2.oraclecloud.com",
    auth_type="basic",
    username="user@company.com",
    password="pass123"
  )
```

Once added, this environment takes precedence over env vars (because it becomes the active environment). You can then remove `FUSION_HOST`, `FUSION_USER`, and `FUSION_PASSWORD` from your MCP client config.

---

## Examples

### Typical Multi-Environment Setup

A common Oracle Fusion implementation with three environments:

```
# 1. Add all environments
→ add_environment(name="dev", fusion_host="https://acme-dev.fa.us2.oraclecloud.com")
→ add_environment(name="uat", fusion_host="https://acme-uat.fa.us2.oraclecloud.com", auth_type="basic", username="uat_user@company.com", password="UatPass123")
→ add_environment(name="prod", fusion_host="https://acme-prod.fa.us2.oraclecloud.com")

# 2. Authenticate to SSO environments (one-time per session)
→ authenticate(environment="dev")    # Opens Chrome → login to dev
→ authenticate(environment="prod")   # Opens Chrome → login to prod
# UAT doesn't need authenticate — it uses basic auth

# 3. Work in dev
→ switch_environment(name="dev")
→ execute_oracle_sql(sql="SELECT * FROM AP_INVOICES_ALL WHERE ROWNUM <= 5")

# 4. Verify same data in UAT
→ switch_environment(name="uat")
→ execute_oracle_sql(sql="SELECT * FROM AP_INVOICES_ALL WHERE ROWNUM <= 5")
```

### Cross-Environment Data Comparison

Compare record counts across environments without switching:

```
→ switch_environment(name="dev")

# Check invoice counts across all environments
→ execute_oracle_sql(sql="SELECT COUNT(*) AS cnt FROM AP_INVOICES_ALL")
  # → dev: 15,432

→ execute_oracle_sql(sql="SELECT COUNT(*) AS cnt FROM AP_INVOICES_ALL", environment="uat")
  # → uat: 12,891

→ execute_oracle_sql(sql="SELECT COUNT(*) AS cnt FROM AP_INVOICES_ALL", environment="prod")
  # → prod: 89,445
```

### Mixed Authentication Methods

Different environments, different auth:

```
# Dev: SSO (interactive development)
→ add_environment(name="dev", fusion_host="https://dev.fa.us2.oraclecloud.com", auth_type="sso")

# UAT: Basic auth (shared test account)
→ add_environment(name="uat", fusion_host="https://uat.fa.us2.oraclecloud.com", auth_type="basic", username="qa_team@company.com", password="QaPass123")

# Prod: SSO (strict security)
→ add_environment(name="prod", fusion_host="https://prod.fa.us2.oraclecloud.com", auth_type="sso")

# Check auth status for each
→ get_auth_status(environment="dev")    # Shows SSO token status
→ get_auth_status(environment="uat")    # Shows basic auth configured
→ get_auth_status(environment="prod")   # Shows SSO token status
```

### MCP Client Configuration

When using environments, the `.mcp.json` setup is simpler — you don't need to put `FUSION_HOST` in the env section at all:

```json
{
  "mcpServers": {
    "fusion-metadata-go": {
      "command": "/path/to/ofmcp",
      "args": ["--db", "/path/to/metadata.db"],
      "env": {
        "LICENSE_PATH": "/path/to/license.json",
        "GEMINI_API_KEY": "AIzaSy..."
      }
    }
  }
}
```

Environments are managed at runtime via the agent conversation. No need to restart the MCP server when adding or switching environments.

---

## Troubleshooting

### "environment X not found"

**Cause:** The environment name doesn't exist in `environments.json`.

**Fix:** Check available environments:
```
→ list_environments
```
Then use the correct name, or add the missing environment:
```
→ add_environment(name="X", fusion_host="https://...")
```

---

### "FUSION_HOST not configured"

**Cause:** No active environment, and no `FUSION_HOST` env var set.

**Fix:**
- Add an environment: `add_environment(name="myenv", fusion_host="https://...")`
- Or set `FUSION_HOST` in your MCP client config's `env` section

---

### "username and password are required when auth_type is 'basic'"

**Cause:** You specified `auth_type="basic"` but didn't provide both `username` and `password`.

**Fix:** Include both credentials:
```
→ add_environment(name="uat", fusion_host="https://...", auth_type="basic", username="user", password="pass")
```

---

### SSO authentication prompt appears for a basic auth environment

**Cause:** The environment's `auth_type` may be set to `sso` (the default).

**Fix:** Update the environment:
```
→ add_environment(name="uat", fusion_host="https://...", auth_type="basic", username="user", password="pass")
```
This overwrites the existing configuration.

---

### Switching environments doesn't seem to take effect

**Cause:** There may be a per-request `environment` parameter overriding the switch.

**Fix:** Check that your tool calls don't include an explicit `environment` parameter. If they do, that always takes priority over the active environment.

---

### Env vars are being used instead of environments

**Cause:** No active environment is set (all environments were removed, or none were added).

**Fix:** Add an environment or re-activate one:
```
→ add_environment(name="dev", fusion_host="https://...")
```
The first environment added becomes active automatically.

---

### Passwords visible in environments.json

**Expected behavior.** For basic auth environments, passwords are stored in plaintext.

**Mitigation:**
1. Restrict file permissions: `chmod 600 environments.json`
2. Use SSO (`auth_type="sso"`) where possible
3. Never commit `environments.json` to version control
4. Consider using service accounts with minimal privileges for basic auth

---

## See Also

- [Environment Variables Reference](./environment-variables.md) — all env vars and MCP client configuration
- [Installation Guide](../../ofrag/docs/installation.md) — initial setup
- [MCP Catalog](./mcp-catalog.md) — full list of available tools
- [Semantic Search Setup](./semantic-search-setup.md) — vector search configuration
