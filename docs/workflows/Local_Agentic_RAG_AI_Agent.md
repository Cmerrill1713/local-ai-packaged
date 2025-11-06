# Local Agentic RAG AI Agent workflow

The `n8n/backup/workflows/Local_Agentic_RAG_AI_Agent.json` export is the
canonical automation delivered with this starter kit. It merges the strongest
ideas from the prior V1, V2, and V3 iterations into a single flow so you can
bootstrap retrieval-augmented agents without juggling multiple workflow files.

## Key capabilities

- **Hybrid vector retrieval** – documents are stored in both Supabase/Postgres
  PGVector and Qdrant. The agent can query either vector store (or both) through
  dedicated tools, giving you the option to experiment with latency and recall
  trade-offs without reconfiguring the workflow.
- **Document metadata management** – imports automatically upsert metadata and
  tabular extracts into Postgres, ensuring the agent can answer structured data
  questions and keep historical context.
- **Local embeddings and models** – Ollama provides embeddings and LLM
  responses, so the entire pipeline continues to run locally.
- **Self-cleaning ingestion** – before new files are chunked, existing records
  are pruned from both Supabase and Qdrant to avoid duplicate vectors.

## Nodes to review after importing

| Node | Purpose |
| --- | --- |
| `Clear Old Vectors` | Removes stale Qdrant vectors for the file being processed. |
| `Qdrant Vector Store Insert` | Writes fresh chunks into the `documents` collection. |
| `Qdrant Vector Store Tool` | Exposes Qdrant as an agent tool alongside the PGVector tool. |
| `Postgres PGVector Store` | Persists chunks inside Supabase/Postgres for long-term storage. |
| `Postgres PGVector Store1` | Registers the Supabase-backed vector store as an agent tool. |

## Credential expectations

When you first open the workflow inside n8n, confirm that these credentials are
set up and mapped to the appropriate nodes:

- `Postgres account` for Supabase/Postgres nodes (`Postgres PGVector Store*`,
  table management nodes, and chat memory).
- `QdrantApi account` for the `Qdrant Vector Store*` nodes. The README lists the
  default URL and notes that you can use any API key for local development.
- `Ollama account` for the embedding and LLM nodes so the workflow can reach the
  Ollama container.

Once those credentials resolve, you can trigger the workflow either via the
`When chat message received` event for Open WebUI integration or by uploading
files into the watched directory. The agent will automatically enrich and store
content in both vector stores, making the deduplicated export the only file you
need to maintain.
