# LangGraph Memory Agent

[![CI](https://github.com/langchain-ai/memory-agent/actions/workflows/unit-tests.yml/badge.svg)](https://github.com/langchain-ai/memory-agent/actions/workflows/unit-tests.yml)
[![Open in LangGraph Studio](https://img.shields.io/badge/Open_in-LangGraph_Studio-00324d.svg)](https://langgraph-studio.vercel.app/templates/open?githubUrl=https://github.com/langchain-ai/memory-agent)

An extensible ReAct-style chatbot that persists user-scoped memories and uses semantic search to personalize future conversations. The agent saves key details across threads and reuses them for continuity, reduced clarification, and more helpful responses.

![Memory Diagram](./static/memory_graph.png)

## Features

- Long-term, user-scoped memory with cross-thread recall.
- Memory metadata: `importance`, `tags`, and `timestamp` captured on write.
- Semantic search tool (`search_memories`) to retrieve recent or query-based memories.
- Tool-driven upsert with conflict-aware updates (`upsert_memory`).
- Configurable model, prompts, and store index (1536‑dim embeddings).

## Quickstart

1. Create and configure environment:

```bash
cp .env.example .env
```

Update `.env` with your provider API key(s) and project name:

```env
LANGSMITH_PROJECT=memory-graph
# Choose provider and set its API key
# ANTHROPIC_API_KEY=...
# OPENAI_API_KEY=...
```

2. Install and run locally (Windows):

```bash
python -m venv venv
venv\Scripts\activate
pip install -e .
pytest -v  # optional: run tests
```

3. Open in LangGraph Studio and chat with the `memory_agent` graph. Share details (name, preferences, etc.), then start a new thread and confirm the agent recalls your information.

![Memories Explorer](./static/memories.png)

## Programmatic Use

```python
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.store.memory import InMemoryStore
from memory_agent.context import Context
from memory_agent.graph import builder

store = InMemoryStore()
graph = builder.compile(store=store, checkpointer=InMemorySaver())

await graph.ainvoke(
    {"messages": [("user", "Hi, I'm Alice and I love pizza.")]},
    {"thread_id": "thread"},
    context=Context(user_id="alice"),
)
```

## How It Works

1. The agent retrieves relevant memories via semantic search and enriches the system prompt.
2. The model responds and may call tools.
3. Tool calls are executed (e.g., `upsert_memory`, `search_memories`) and results are fed back to the conversation.

## Evaluation

- Integration tests in `tests/integration_tests/test_graph.py` simulate short/medium/long conversations and verify memories are stored.
- Tests run under PyTest; optional LangSmith instrumentation is available for tracking and analysis.

Run locally:

```bash
pytest -v
```

## Customization

- Memory schema: extend `importance`, `tags`, or add custom fields in `tools.py`.
- Tools: add new functions and bind them in `graph.py` to enable model-initiated actions.
- Model selection: set `Context.model` via environment or code to choose provider/model.
- Prompts: edit `src/memory_agent/prompts.py` to tune system behavior.
