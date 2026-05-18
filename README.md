# Podcast MCP Server

> **Part 2 of the "How to build your podcast AI assistant" series.**
> Part 1 covered building the podcast database. This repo covers the MCP server that exposes it to any AI agent.

An [MCP (Model Context Protocol)](https://modelcontextprotocol.io) server that lets any compatible LLM (Claude, Mistral, Gemini…) query a ChromaDB podcast knowledge base using RAG — without any change to the server when you switch models.

---

## How it works

```
AI Agent (Claude / Mistral / Gemini)
        │  MCP tool call
        ▼
  Podcast MCP Server          ← this repo
        │  vector search
        ▼
  ChromaDB podcast database   ← built in Part 1
```

The agent calls the `query_podcast_advices` tool with a natural-language question. The server embeds the query via HuggingFace, searches the ChromaDB collection, and returns the top 5 matching podcast excerpts with metadata (title, episode, Spotify URL, similarity score).

---

## Prerequisites

- Python 3.12+ (or Docker)
- A [HuggingFace token](https://huggingface.co/settings/tokens) (free)
- A pre-built ChromaDB podcast database (see Part 1)

---

## Quick start

### 1. Clone and configure

```bash
git clone <repo-url>
cd BKHK-mcp-server
cp .env.example .env
```

Edit `.env` and fill in your values:

```env
HF_TOKEN=your_hf_token_here                                          # required
DB_PATH=/app/podcast_db_local_fr                                     # path to your ChromaDB database
HF_MODEL=sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2 # embedding model
PORT=8080                                                            # server port
```

> Use a multilingual model (like the default) if your podcast content is not in English.

### 2. Run with Docker (recommended)

```bash
# Build the image
docker build -t podcast-mcp-server .

# Run with your .env file and your local database mounted
docker run --rm \
  --env-file .env \
  -e DB_PATH=/app/podcast_db_local_fr \
  -v ./podcast_db_local_fr:/app/podcast_db_local_fr \
  -p 8080:8080 \
  podcast-mcp-server
```

Or with Docker Compose:

```bash
docker compose up
```

### 3. Run locally

```bash
# Install dependencies (requires uv)
uv sync

# Start the server
uv run src/podcast_mcp_server.py
```

The server starts at `http://localhost:8080`.

---

## Connecting an AI agent

Point any MCP-compatible agent to:

```
http://localhost:8080/mcp
```

### Example: Claude Desktop (`claude_desktop_config.json`)

```json
{
  "mcpServers": {
    "podcast": {
      "url": "http://localhost:8080/mcp"
    }
  }
}
```

### System prompt structure

To get the best results, give your agent a system prompt built around four questions:

| Block | Purpose |
|---|---|
| **Who is the agent?** | Persona, tone, and communication style |
| **What does it know?** | Scope of the knowledge base — prevents hallucination |
| **When does it stop?** | What the agent refuses and when it redirects |
| **How does it answer?** | Response format: validate first, advise, stay concise, offer to go deeper |

---

## Environment variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `HF_TOKEN` | Yes | — | HuggingFace API token |
| `DB_PATH` | No | `/app/podcast_db_local_fr` | Path to the ChromaDB database directory |
| `HF_MODEL` | No | `sentence-transformers/all-MiniLM-L6-v2` | HuggingFace embedding model |
| `PORT` | No | `8080` | Port the MCP server listens on |

---

## Project structure

```
.
├── src/
│   ├── podcast_mcp_server.py   # MCP server and ChromaDB query logic
│   └── hf_embeddings.py        # HuggingFace embedding function for ChromaDB
├── .env.example                # Environment variable template
├── Dockerfile
├── docker-compose.yml
└── pyproject.toml
```

---

## Key benefits

**LLM-agnostic** — The MCP protocol decouples the agent from the server. Swap Claude for Mistral or any other model without touching this codebase.

**Separation of concerns** — The database (Part 1) and the server (this repo) are independent. Update the knowledge base without redeploying the server, and vice versa.

**Persona through prompting** — The agent's voice and behaviour are defined entirely in the system prompt, not in server code. Reconfigure for a new podcast or a new client by changing the prompt only.

---

## What's next

Part 3 — How to host this podcast AI assistant on AWS.
