# CLAUDE.md — Agent Zero

## Project Overview

Agent Zero is an open-source, general-purpose AI agent framework. It is a personal assistant that grows and learns through usage, featuring a hierarchical multi-agent architecture, persistent memory, browser automation, code execution, and a real-time streaming web UI.

## Tech Stack

- **Backend**: Python 3.x, Flask (async), LiteLLM (multi-provider LLM abstraction)
- **Frontend**: HTML/JS with Alpine.js for state management
- **Vector DB**: FAISS (via sentence-transformers) for semantic memory search
- **Browser Automation**: browser-use + Playwright
- **Containerization**: Docker (optional, for sandboxed code execution)
- **Protocols**: MCP (Model Context Protocol), A2A (Agent-to-Agent)

## Directory Structure

```
├── agent.py              # Core Agent and AgentContext classes
├── models.py             # LLM model configuration and provider setup
├── initialize.py         # Framework initialization and config
├── run_ui.py             # Flask web server entry point
├── requirements.txt      # Python dependencies
├── agents/               # Agent profiles (default, developer, researcher, hacker)
├── python/
│   ├── api/              # 50+ REST API endpoint handlers
│   ├── tools/            # 22 agent tools (code exec, browser, memory, etc.)
│   ├── extensions/       # 23 hook-based extension categories
│   └── helpers/          # 70+ utility modules
├── prompts/              # Modular system prompt files (Markdown)
├── webui/                # Frontend (HTML, JS, CSS)
├── memory/               # Persistent agent memory storage (gitignored)
├── knowledge/            # Knowledge base files
├── instruments/          # Custom user-defined functions/scripts
├── logs/                 # HTML chat logs (gitignored)
├── conf/                 # Config files (model_providers.yaml)
├── docker/               # Docker configuration and compose files
└── tmp/                  # Runtime settings and temp data (gitignored)
```

## Key Entry Points

- **`run_ui.py`** — Main entry point. Starts the Flask server, registers API handlers, initializes agents and MCP.
- **`agent.py`** — Core `Agent` and `AgentContext` classes. The agent message loop, tool execution, and subordinate delegation live here.
- **`initialize.py`** — Reads user settings and creates `AgentConfig` with model configs, memory paths, and runtime settings.
- **`models.py`** — `ModelConfig` and `ModelType` definitions; LLM provider configuration.

## Architecture Patterns

- **Extension hook system**: 23 extension categories in `python/extensions/` (e.g., `system_prompt`, `message_loop_*`, `tool_execute_*`, `monologue_*`). Files execute alphabetically by numeric prefix (e.g., `_10_*.py` before `_90_*.py`).
- **Tool-based execution**: Tools in `python/tools/` inherit a base `Tool` class, receive `args` dict, and return `Response` objects.
- **Hierarchical agents**: Agents can spawn subordinates via `call_subordinate` tool. Results propagate back up the hierarchy.
- **API auto-registration**: API handlers in `python/api/` are auto-discovered and registered as Flask routes by filename.
- **Async-first**: Uses `async/await` throughout with `DeferredTask` for background work and `nest-asyncio` for nested event loops.

## Configuration

- **`.env`** — API keys and runtime config (not committed; see root for any example env patterns).
- **`conf/model_providers.yaml`** — LLM provider definitions (Anthropic, OpenAI, Groq, Ollama, etc.).
- **`tmp/settings.json`** — Runtime user settings (model selection, API keys, behaviors). Persisted automatically.
- **Agent profiles** — Each profile in `agents/` can override prompts, tools, and behaviors.

## Running the App

```bash
# Install dependencies
pip install -r requirements.txt

# Run the web UI (default: http://localhost:80)
python run_ui.py
```

Or via Docker:
```bash
cd docker/run
docker-compose up
```

Environment variables:
- `WEB_UI_HOST` — Bind address (default: `localhost`)
- `WEB_UI_PORT` — Port (default: `80`)
- `AUTH_LOGIN` / `AUTH_PASSWORD` — Optional basic auth for the UI
- `FLASK_SECRET_KEY` — Flask session secret (auto-generated if unset)

## Development Notes

- **Adding a new tool**: Create a Python file in `python/tools/` following the existing pattern (inherit base Tool class). It will be auto-discovered.
- **Adding a new API endpoint**: Create a Python file in `python/api/` with a class inheriting `ApiHandler`. It auto-registers as `/<filename>`.
- **Adding extensions**: Place numbered Python files in the appropriate `python/extensions/<category>/` folder.
- **Prompts are Markdown**: System prompts live in `prompts/` as `.md` files and support placeholder replacement.
- **Memory/knowledge/instruments/logs/tmp** are gitignored — do not expect these to persist across clones.

## Dependencies (Key Packages)

| Package | Purpose |
|---------|---------|
| `flask[async]` | Web server |
| `litellm` (via langchain) | Multi-provider LLM calls |
| `faiss-cpu` | Vector similarity search |
| `sentence-transformers` | Text embeddings |
| `browser-use` + `playwright` | Browser automation |
| `docker` | Container management for sandboxed execution |
| `paramiko` | SSH for remote code execution |
| `kokoro` | Text-to-speech |
| `openai-whisper` | Speech-to-text |
| `fastmcp` + `mcp` | MCP protocol support |
| `pydantic` | Data validation |

## Syncing with Upstream

This is a fork of `agent0ai/agent-zero`. To pull the latest changes from upstream:

```bash
git pull upstream main && git push
```

The `upstream` remote is already configured pointing to `https://github.com/agent0ai/agent-zero.git`.
