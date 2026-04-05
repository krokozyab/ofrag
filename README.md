<h1 align="center">ofrag — MCP Server for Oracle Fusion Cloud</h1>

<p align="center">
  <strong>Give your AI agent direct access to Oracle Fusion ERP data. Ask questions in plain English — get SQL results in seconds.</strong>
</p>

<p align="center">
  <a href="https://oraclefusionsql.com"><img src="https://img.shields.io/badge/Website-oraclefusionsql.com-orange?style=for-the-badge" alt="Website"></a>
</p>

<p align="center">
  <a href="#-the-magic-moment">Demo</a> · 
  <a href="#-what-is-ofrag">What is it</a> · 
  <a href="#-tool-catalog">Tools</a> · 
  <a href="#-quick-start">Quick Start</a> · 
  <a href="#-documentation">Docs</a>
</p>

---

## The Problem

Oracle Fusion Cloud has **25,000+ tables** across Finance, SCM, HCM, and Procurement. Finding the right table, understanding join relationships, and writing correct SQL takes hours of digging through Oracle docs. Even experienced consultants spend 15+ minutes per query cycle: log into BI Publisher → create data model → write SQL → debug → re-run.

**ofrag eliminates this entirely.** Your AI agent discovers tables semantically, validates SQL against a local schema cache, and executes queries against live Fusion data — all in one conversation.

## ✨ The Magic Moment

> **You:** "Show me the top 5 unpaid invoices for vendor Acme Corp generated last month."
>
> **Claude + ofrag:**
> 1. **Discovers** the right tables via semantic search (`AP_INVOICES_ALL`, `POZ_SUPPLIERS`, `HZ_PARTIES`)
> 2. **Understands** "unpaid" means `PAYMENT_STATUS_FLAG = 'N'`
> 3. **Validates** the SQL against local metadata — catches errors before they hit production
> 4. **Executes** the query against your live Oracle Fusion instance
> 5. **Returns** a formatted table with invoice numbers, amounts, and dates
>
> **Total time: ~4 seconds.**

No BI Publisher. No data model. No OTBI subject areas. Just a question and an answer.

## 🧠 What is ofrag

ofrag is an **MCP (Model Context Protocol) server** — an open standard that lets AI agents use external tools. It gives Claude, OpenAI, Gemini, and any MCP-compatible LLM structured access to your Oracle Fusion Cloud data.

```
┌──────────────────────────────────────────────────────────────┐
│                      Your AI Agent                           │
│           Claude · OpenAI · Gemini · Claude Code             │
├──────────────────────────────────────────────────────────────┤
│                    ofrag (MCP Server)                        │
│                                                              │
│  ┌─────────────┐  ┌──────────────┐  ┌─────────────────────┐  │
│  │  Semantic   │  │  SQL Linter  │  │  Business Process   │  │
│  │  Search     │  │  (AST-based) │  │  Mapper             │  │
│  └──────┬──────┘  └──────┬───────┘  └──────────┬──────────┘  │
│         │                │                     │             │
│  ┌──────▼──────────────────────────────────────▼──────────┐  │
│  │           Local DuckDB Metadata Cache                  │  │
│  │          25,000+ tables · 240K+ descriptions           │  │
│  └────────────────────────┬───────────────────────────────┘  │
│                           │ Only validated queries           │
├───────────────────────────▼──────────────────────────────────┤
│                  Oracle Fusion Cloud                         │
│            BI Publisher SOAP · REST APIs                     │
└──────────────────────────────────────────────────────────────┘
```

**Key principle:** All schema exploration happens locally — zero queries to production during discovery. Only final, validated SQL reaches your live Fusion instance.

## 🔑 Why ofrag Has No Alternatives


| | ofrag + AI Agent | Manual BI Publisher | OTBI | Proprietary SQL Tools |
|---|:---:|:---:|:---:|:---:|
| **Natural language queries** | ✅ Ask in plain English | ❌ Write SQL manually | ❌ Drag-and-drop only | ❌ Write SQL manually |
| **Schema discovery** | ✅ Semantic search across 25K+ tables | ❌ Read Oracle docs | ❌ Limited subject areas | ✅ Table browser |
| **SQL validation before execution** | ✅ AST parsing against local cache | ❌ Trial and error | N/A | ❌ |
| **Cross-module table relationships** | ✅ AI traces AP→XLA→GL chains | ❌ Manual knowledge | ❌ Single subject area | ❌ |
| **Business process mapping** | ✅ `business_process_map`, `scenario_mapper` | ❌ | ❌ | ❌ |
| **Multi-environment switching** | ✅ DEV/SIT/UAT/PROD in one session | ❌ Separate logins | ❌ Separate logins | Partial |
| **REST API access** | ✅ Built-in universal REST client | ❌ | ❌ | ❌ |
| **Production safety** | ✅ Local-first, validated queries only | ⚠️ Direct execution | ✅ Sandboxed | ⚠️ Direct execution |

> **The real difference:** Other tools help you *write* SQL. ofrag lets you *skip* writing SQL — the AI does it for you, correctly, using verified metadata.

## 🛠 Tool Catalog

ofrag exposes **30+ tools** to your AI agent, organized by function:

### Discovery & Search
| Tool | What it does |
|---|---|
| `semantic_search` | Find tables/columns by business meaning ("unpaid invoices", "employee compensation") using vector embeddings across 240K+ descriptions |
| `search_identifiers` | Find tables/columns by Oracle name pattern (`AP_INVOICES`, `VENDOR_ID`) |
| `search_descriptions` | Exact keyword search in table/column descriptions |
| `list_tables` | Browse tables by module with filters |
| `list_columns` | Get all columns, types, and descriptions for a table |
| `index_info` | Index definitions for a table |
| `table_overview` | Quick summary: column count, purpose, module |

### SQL & Execution
| Tool | What it does |
|---|---|
| `execute_oracle_sql` | Execute live SQL against Oracle Fusion, results inline |
| `execute_oracle_sql_to_file` | Execute SQL, save results to local file (for large datasets) |
| `lint_sql` | Parse SQL into AST, validate against cached schema — catches errors before execution |
| `suggest_sql` | Auto-complete partial SQL using metadata cache |
| `raw_select` | Query the local DuckDB metadata cache directly |

### Business Intelligence
| Tool | What it does |
|---|---|
| `module_summary` | Overview of a Fusion module — purpose, key tables, typical queries |
| `business_process_map` | Map a business process to its tables, stages, and key columns |
| `scenario_mapper` | Map a business intent ("month-end close") to recommended tables and metrics |
| `relationship_map` | Show inbound/outbound JOIN relationships for any table |
| `cross_module_analyzer` | Analyze how a table connects across Fusion modules |
| `integration_flow_mapper` | Trace end-to-end data flow: upstream feeders → table → downstream consumers |
| `process_catalog` | Full business process catalog with all stages and tables |

### REST API
| Tool | What it does |
|---|---|
| `rest_call` | Make HTTP REST API calls to Oracle Fusion, results inline |
| `rest_call_to_file` | REST API calls with results saved to file |

### Fusion Configuration
| Tool | What it does |
|---|---|
| `describe_flexfield` | Describe Descriptive Flexfields (DFF) — contexts, segments, value sets |
| `lookup_values` | Get all values for a Fusion lookup type (LOV) |
| `profile_values` | Get profile option values at site/product/user levels |

### BI Publisher & ESS
| Tool | What it does |
|---|---|
| `run_bi_report` | Execute a BI Publisher report with parameters, results inline |
| `run_bi_report_to_file` | Execute BI Publisher report, save output to file |
| `submit_ess_job` | Submit an ESS job (batch processes, imports, exports) |
| `get_ess_job_status` | Check ESS job status |

### Environment Management
| Tool | What it does |
|---|---|
| `authenticate` | Browser-based SSO login to Oracle Fusion |
| `get_auth_status` | Check current authentication state |
| `add_environment` | Add/update a named environment (DEV, SIT, UAT, PROD) |
| `switch_environment` | Switch active environment instantly |
| `list_environments` | List all configured environments |
| `compare_environments` | Compare configuration data between two environments |

## 🚀 Quick Start

### 1. Get the binary
Download from [Releases](https://github.com/krokozyab/ofrag/releases) (Windows and macOS).

### 2. Follow the setup guide

👉 [**Installation & Setup Guide**](docs/installation.md) — step-by-step: licensing, BI Publisher report deployment, MCP client configuration (Claude Desktop, Gemini, Claude Code), authentication setup.

### 3. Start talking to your ERP
> "What tables store AP invoice data?"  
> "Show me all overdue invoices for Acme Corp"  
> "How are accounts payable balances linked to the general ledger?"  
> "Compare the supplier count between DEV and PROD"

## 💡 Use Cases

**Ad-hoc data analysis** — Ask "show me revenue by business unit for Q4" and get results in seconds. No OTBI report creation, no BI Publisher data model, no waiting.

**Implementation troubleshooting** — During Fusion implementation, quickly verify data: "are there any AP invoices without distributions?", "show me GL journal entries that haven't been transferred from XLA".

**Data reconciliation** — "Compare AP subledger totals against GL balances for period DEC-24". The AI traces the full AP → XLA → GL chain and identifies discrepancies.

**Schema exploration** — "What tables store employee compensation data?" The AI uses semantic search to find relevant HCM tables, shows their relationships, and explains the data model.

**Integration development** — Building OIC integrations? Ask "what REST endpoints exist for AP invoices?" or "what's the FBDI template for importing journals?" — ofrag finds it.

**EBS → Fusion migration** — "What's the Fusion equivalent of AP_INVOICES_ALL in EBS?" The AI maps EBS tables to Fusion tables using semantic search and module knowledge.

**Report development** — Before building a BI Publisher report, prototype your SQL through conversation. The AI validates and optimizes your queries using the local schema cache.

## ⚠️ Limitations

- **Read-only SQL** — `SELECT` queries only through BI Publisher SOAP. Oracle Fusion does not permit write access through this layer.
- **Security** — Ensure usage complies with your organization's security policies. Credentials are stored locally and never transmitted to third parties.

## 📄 Documentation

| Guide                                                     | Description |
|-----------------------------------------------------------|---|
| [Installation & Setup](docs/installation.md)              | Detailed guide on how to install, configure, and get the binary running |
| [Full MCP Server Reference](docs/README.md)               | Complete reference for configuration, authentication, variables and tools |
| [Multi-Environment Management](docs/multi-environment.md) | Configure and switch between dev, UAT, prod environments interactively |
| [Semantic Search Setup](docs/semantic-search-setup.md)    | How to enable vector search and REST API catalog for full-powered semantic discovery |

## 🌐 Ecosystem

ofrag is part of the Oracle Fusion open-source ecosystem:

| Project | What it does | Link |
|---|---|---|
| **ofrag** | MCP Server — AI-powered queries via Claude, OpenAI, Gemini | [GitHub](https://github.com/krokozyab/ofrag) |
| **OFJDBC** | JDBC driver — SQL access from DBeaver, IntelliJ, JVM apps, ETL pipelines | [GitHub](https://github.com/krokozyab/ofjdbc) |

## 📫 Contact

- **Website:** [oraclefusionsql.com](https://oraclefusionsql.com)
- **GitHub Issues:** [krokozyab/ofrag/issues](https://github.com/krokozyab/ofrag/issues)
- **Email:** sergey.rudenko.ba@gmail.com
- **LinkedIn:** [Sergey Rudenko](https://www.linkedin.com/in/sergey-rudenko-ba/)

---

<p align="center">
  If ofrag saved you time, consider leaving a ⭐
  <br><br>
  <strong>The only MCP server for Oracle Fusion Cloud.</strong> Built by an Oracle Fusion consultant who got tired of BI Publisher.
</p>
