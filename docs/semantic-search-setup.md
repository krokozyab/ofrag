# Setting Up Semantic Search

`semantic_search` is the primary discovery tool in ofrag. Out of the box it uses keyword matching, synonym expansion, and business process cross-referencing. The unified `metadata.db` includes **vector embeddings** for 200K+ SQL table/column descriptions and **REST API resource discovery** across 500+ Oracle Fusion endpoints. Adding a free Gemini API key enables vector similarity search at query time.

---

## What You Get

| Component | What it adds | Size |
|-----------|-------------|------|
| `metadata.db` | Unified database: schema metadata + vector embeddings + REST catalog | ~800 MB |
| `GEMINI_API_KEY` | Converts your search query into a vector at query time (one lightweight API call per search) | Free |

Both are included in every release. Without the API key, `semantic_search` works via keyword + fuzzy matching only — vector similarity is skipped.

---

## Setup

### 1. Download release files

Download **`ofmcp`** and **`metadata.db`** from the [latest release](https://github.com/krokozyab/ofrag/releases). Place them in the same folder:

```
/your/path/
├── ofmcp              ← binary
├── metadata.db        ← unified database (schema + embeddings + REST catalog)
└── license.json       ← license file
```

No separate `embeddings.db` or `rest_catalog.db` needed — everything is in `metadata.db`.

### 2. Get a free Gemini API key

Vector search requires a Gemini API key **only at query time** — to convert your search text into a vector (one API call per search, ~100ms).

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Create a free API key (30 seconds, no billing required)

Without the key, keyword search still works on all data. You just won't get vector similarity results.

### 3. Add the key to your MCP config

**Claude Desktop** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "fusion-metadata": {
      "command": "/path/to/ofmcp",
      "args": ["--db", "/path/to/metadata.db"],
      "env": {
        "FUSION_HOST": "https://your-instance.oraclecloud.com",
        "GEMINI_API_KEY": "AIzaSy...your-key"
      }
    }
  }
}
```

**Claude Code** (`.mcp.json` in project root):

```json
{
  "mcpServers": {
    "fusion-metadata-go": {
      "command": "/path/to/ofmcp",
      "args": ["--db", "/path/to/metadata.db"],
      "env": {
        "FUSION_HOST": "https://your-instance.oraclecloud.com",
        "GEMINI_API_KEY": "AIzaSy...your-key"
      }
    }
  }
}
```

---

## How It Works

```
Your query: "supplier invoices"
        │
        ▼
┌──────────────────────────────────────┐
│  1. SQL keyword search               │──► tables/columns by name and remarks
│  2. SQL vector similarity            │──► tables/columns by description meaning
│  3. REST resource keyword search     │──► REST resources by name and title
│  4. REST vector similarity           │──► REST resources & attributes by meaning
│  5. Adaptive cutoff (knee-point)     │──► remove noise based on score curve
│  6. Description keyword boost        │──► boost tables with query terms in description
│  7. Blend scores, sort, return       │──► combined results with SQL + REST sections
└──────────────────────────────────────┘
```

Scores are blended from keyword match, fuzzy string matching, vector similarity, and description keyword presence. SQL results get additional boosts from business process definitions, module clustering, and relationship density.

The **knee-point algorithm** adaptively determines where to cut off vector search results by analyzing the inflection point in the similarity score curve — instead of a fixed threshold, it adapts to each query.

---

## Configuration Reference

| Setting | Where | Description |
|---------|-------|-------------|
| `GEMINI_API_KEY` | MCP config `env` | Free Gemini API key for vector search |
| `--gemini-api-key` | MCP config `args` | Alternative to env variable |
| `GEMINI_EMBEDDING_MODEL` | MCP config `env` | Override model (default: `gemini-embedding-2-preview`) |

The API key is resolved in order: `--gemini-api-key` flag → `GEMINI_API_KEY` env → `GEMINI_API_KEY` in `.mcp.json`.

---

## Troubleshooting

### Vector search not improving results
Vector search works best for natural language queries ("employee salary history", "money received from customers"). For Oracle identifier lookups ("AP_INVOICES", "VENDOR_ID"), use `search_identifiers` instead.

### No REST section in results
REST results appear only when they score high enough to make the top N. Try increasing `limit` or use a query that matches REST resource names directly.

### Everything works but no vector results (only keyword)
Check that `GEMINI_API_KEY` is set. Without it, keyword search still works but vector similarity is skipped.
