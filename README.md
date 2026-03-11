# Oracle Fusion RAG Engine (`ofrag`) MCP server

**Start chatting with your ERP.**
[![GitHub Downloads](https://img.shields.io/github/downloads/krokozyab/ofrag/total?style=for-the-badge&logo=github)](https://github.com/krokozyab/ofrag/releases)
This is a specialized **Retrieval-Augmented Generation (RAG) engine designed for LLMs** (like Claude). It empowers your AI agent to retrieve schema context locally, validate intent via semantic parsing, and fetch live business data directly from Oracle Fusion Cloud.

---

## The Magic Moment

> **User:** "Show me the top 5 unpaid invoices for vendor 'Acme Corp' generated last month."
>
> **Claude + ofrag:**
> 1.  *Understands* "unpaid" means `PAYMENT_STATUS_FLAG = 'N'`.
> 2.  *Lints* the SQL against local metadata to ensure the table `AP_INVOICES_ALL` exists.
> 3.  *Executes* the query directly against your live Oracle Fusion instance.
> 4.  *Returns* a perfectly formatted markdown table.
>
> **Total Time:** 4 seconds.

---

## Why `ofrag`?

### 🚀 Velocity for Experts, Access for Everyone
We shift the paradigm of data access.
*   **For the Analyst:** Data is no longer locked in technical silos. If you can type a question, you can get an answer.
*   **For the Consultant:** This is your productivity multiplier. Skip the 15-minute cycle of logging into BI Publisher, creating data models, and debugging XML. Execute, iterate, and verify complex queries in seconds. It shifts your focus from "how do I extract this?" to "what does this mean?"

### 🧠 Agentic SQL Synergy (RAG Augmentation)
This isn't just a "dumb pipe" for SQL—it is a cognitive force multiplier for your AI.
*   **The Workflow:** When you ask the LLM to fix or improve a SQL query, it utilizes the `lint_sql` tool.
*   **Deep Parsing:** The tool parses the provided SQL into an Abstract Syntax Tree (AST) locally, checking it against the Oracle schema cache.
*   **Multiplied Intelligence:** Instead of guessing, the LLM receives precise structural feedback (e.g., "The column `VENDOR_NAME` does not exist in table `AP_INVOICES_ALL`"). This **augments** the LLM's context, allowing it to perform highly accurate SQL repairs and optimizations that would otherwise be hallucinations and tokens waste.

### 🌐 Universal REST — Self-Discovering API Agent
The `rest_call` tool isn’t limited to a handful of hardcoded endpoints — it turns your AI into a **universal Oracle Fusion REST client**.
*   **Schema-Free Operations:** Oracle Fusion REST endpoints are largely self-describing. Claude can call `/describe` on any resource to understand its structure on the fly, then perform the actual operation — creates, updates, queries — without needing hardcoded schemas.
*   **Full CRUD:** GET, POST, PUT, PATCH, DELETE — the agent handles any HTTP method with managed SSO authentication, so it can read invoices, create suppliers, update PO lines, or call custom endpoints.
*   **SQL Fallback:** When the agent needs deeper data insight (e.g., checking table relationships, resolving IDs, or aggregating across modules), it seamlessly falls back to direct SQL via the existing SQL tools. REST for operations, SQL for analytics — best of both worlds.
*   **Bulk Export:** For large datasets, `rest_call_to_file` automatically paginates through Oracle Fusion’s `{items, hasMore, next}` pattern and merges everything into a single local file.

### ⚡ Local Intelligence (RAG Retrieval)
We respect your environment by running sophisticated analysis **locally**.
*   **Advanced Introspection Tools:** It’s not just a cache; it’s a search engine. `ofrag` ships with a suite of local tools—**Fuzzy Search** (`search_identifiers`), **Semantic Discovery** (`semantic_search`), and **Module Context Analyzers** (`module_summary`)—that traverse your schema instantly.
*   **Zero-Latency Reasoning:** When the AI explores your schema to understand how `AP_INVOICES` relates to `PO_HEADERS`, it uses these local tools to "think" about your data structure.
*   **Less Pressure:** Your Oracle Fusion database receives only the final, polished queries—never the heavy exploratory workload.

---

## Documentation

*   [**Installation & Setup**](docs/installation.md) - Detailed guide on how to install, configure, and get the binary running.
*   [**Detailed MCP Server Docs**](docs/README.md) - Complete reference for configuration, authentication, variables and tools.

---

