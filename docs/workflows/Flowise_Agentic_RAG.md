# Flowise Agentic RAG chatflow

The exported chatflow at `flowise/Web Search + Local Agent Chatflow.json`
combines local retrieval with external tools so you can explore agentic RAG
without any dependence on n8n or Supabase.

## Highlights

- **Vector search with Qdrant** – chunks are written to and queried from the
  `documents` collection. Swap in your own collection name by updating the node
  settings after import.
- **Web results via Brave Search** – the `Brave Search` tool enriches answers
  with fresh context. Provide your API key in the tool node before running.
- **Slack tooling** – helper tools post to incoming webhooks and fetch recent
  conversations directly from the Slack API (provide your webhook URL and bot
  token in the node variables).
- **Local knowledge authoring** – the `Create Markdown Document` tool writes new
  notes into `/data/shared` inside the Flowise container so you can quickly add
  context for future retrieval runs.

## After importing

1. In Flowise → Tools import the custom tool JSON exports found in the
   `flowise/` directory (`create_markdown_doc`, `send_slack_message_via_webhook`,
   `fetch_slack_conversation`, and `list_qdrant_collections`).
2. Open Flowise → Chatflows → *Import* and select the chatflow JSON file.
3. Set environment variables/credentials referenced in node descriptions:
   - `BRAVE_API_KEY` for the search node.
   - `SLACK_WEBHOOK_URL` and `SLACK_BOT_TOKEN` for the Slack helpers.
   - `QDRANT_URL`/`QDRANT_API_KEY` if you have customised the Qdrant service.
4. Trigger the `Qdrant Upsert` branch by uploading a document with the `Document
   Loader` node or by seeding content through the `Create Markdown Document`
   tool.
5. Chat with the agent through the Flowise UI. The Buffer Memory node keeps the
   transcript grounded while the agent dynamically selects tools.

## Extending the chatflow

- Swap the Ollama model for a larger context model if you have the VRAM.
- Add additional tools (Git, HTTP requests, local shell commands) to extend the
  agent’s reach.
- Wire the Langfuse Tracer component into the chain to capture run metadata in
  the Langfuse dashboard bundled with this repository.

With Qdrant as the single source of truth, the Flowise chatflow is far easier
to maintain and deploy than the earlier n8n/Supabase variants.
