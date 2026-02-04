# Oracle Fusion RAG Engine (`ofrag`) MCP server

**Start chatting with your ERP.**

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

### ⚡ Local Intelligence (RAG Retrieval)
We respect you environment by running sophisticated analysis **locally**.
*   **Advanced Introspection Tools:** It’s not just a cache; it’s a search engine. `ofrag` ships with a suite of local tools—**Fuzzy Search** (`search_identifiers`), **Semantic Discovery** (`semantic_search`), and **Module Context Analyzers** (`module_summary`)—that traverse your schema instantly.
*   **Zero-Latency Reasoning:** When the AI explores your schema to understand how `AP_INVOICES` relates to `PO_HEADERS`, it uses these local tools to "think" about your data structure.
*   **Less Pressure:** Your Oracle Fusion database receives only the final, polished queries—never the heavy exploratory workload.

---

## Documentation

*   [**Installation & Setup**](docs/installation.md) - Detailed guide on how to install, configure, and get the binary running.
*   [**Detailed MCP Server Docs**](docs/README.md) - Complete reference for configuration, authentication, variables and tools.

---

