# Setting Up Semantic Search

`semantic_search` is the primary discovery tool in ofrag. Out of the box it uses keyword matching, synonym expansion, and business process cross-referencing. By adding two optional database files and a free API key, you unlock **vector similarity search** across 25K+ SQL table and view descriptions and **REST API resource discovery** across 500+ Oracle Fusion endpoints.

---

## What You Get

| File | What it adds | Size |
|------|-------------|------|
| `embeddings.db` | Vector search on SQL table descriptions — finds tables by meaning, not just name | ~54 MB |
| `rest_catalog.db` | REST API resources and attributes in search results — shows endpoint URLs alongside SQL tables | ~46 MB |
| `GEMINI_API_KEY` | Converts your search query into a vector at query time (one lightweight API call per search) | Free |

All three are optional. Without them, `semantic_search` works as before — keyword + fuzzy matching only.

---

## Setup

### 1. Download the database files

Both files are included in every [GitHub release](https://github.com/user/of_rag/releases). Download them and place next to your `metadata.db`:

```
/your/path/
├── ofmcp              ← binary
├── metadata.db        ← required (schema cache)
├── embeddings.db      ← optional (SQL vector search)
└── rest_catalog.db    ← optional (REST API catalog)
```

No flags or config changes needed — the server auto-detects both files on startup.

### 2. Get a free Gemini API key

Vector search requires a Gemini API key **only at query time** — to convert your search text into a vector (one API call per search, ~100ms).

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Create a free API key (30 seconds, no billing required)

Without the key, `embeddings.db` and `rest_catalog.db` are still loaded — keyword search works on both. You just won't get vector similarity results.

### 3. Add the key to your MCP config

**Claude Desktop** (`claude_desktop_config.json`):

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

### 4. Verify

Check the server logs on startup. You should see:

```
Vector search enabled (embeddings: /your/path/embeddings.db)
REST catalog loaded: /your/path/rest_catalog.db
```

Run a test search:

```
semantic_search("supplier invoices")
```

Expected output — two sections:

```
### SQL Tables & Columns
| Kind  | Schema | Table           | Column | Score |
| table | FUSION | AP_INVOICES_ALL |        | 66    |
| ...   |        |                 |        |       |

### REST API Resources
| Kind          | Resource                          | Attribute | Endpoint                   | Score |
| rest_resource | chargeLinesForInvoiceAssociations |           | /fscmRestApi/resources/... | 56    |
| ...           |                                   |           |                            |       |
```

If only the SQL section appears, REST results may have scored below the limit — try `limit: 30`.

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
│  4. REST vector similarity           │──► REST resources & attributes by description meaning
│  5. Blend scores, sort, return       │──► combined results in two sections
└──────────────────────────────────────┘
```

Scores are blended from keyword match, fuzzy string matching, and vector similarity. SQL results get additional boosts from business process definitions, module clustering, and relationship density. The query embedding is computed once and reused for both SQL and REST vector search.

---

## Configuration Reference

| Setting | Where | Description |
|---------|-------|-------------|
| `GEMINI_API_KEY` | MCP config `env` | Free Gemini API key for vector search |
| `--embeddings-db` | MCP config `args` | Custom path for embeddings.db (default: next to metadata.db) |
| `--gemini-api-key` | MCP config `args` | Alternative to env variable |
| `GEMINI_EMBEDDING_MODEL` | MCP config `env` | Override model (default: `gemini-embedding-2-preview`) |

The API key is resolved in order: `--gemini-api-key` flag → `GEMINI_API_KEY` env → `GEMINI_API_KEY` in `.mcp.json`.

---

## Troubleshooting

### Vector search not improving results
Vector search works best for natural language queries ("employee salary history", "money received from customers"). For Oracle identifier lookups ("AP_INVOICES", "VENDOR_ID"), use `search_identifiers` instead.

### "Embeddings DB not found" in logs
Make sure `embeddings.db` is in the same directory as `metadata.db`, or use `--embeddings-db` to specify a custom path.

### "REST catalog loaded" doesn't appear
Make sure `rest_catalog.db` is in the same directory as `metadata.db`. There is no separate flag for it.

### No REST section in results
REST results appear only when they score high enough to make the top N. Try increasing `limit` or use a query that matches REST resource names directly.

### Everything works but no vector results (only keyword)
Check that `GEMINI_API_KEY` is set. Without it, keyword search still works on both databases but vector similarity is skipped.
