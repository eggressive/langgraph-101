# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LangGraph 101 is an educational repository containing hands-on tutorials for learning LangChain v1 and LangGraph v1. The repository includes Jupyter notebooks for learning and standalone agent implementations that can be deployed as services.

## Development Setup

### Environment Setup
```bash
# Install uv package manager
pip install uv

# Install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate

# Copy environment template and configure API keys
cp .env.example .env
# Edit .env to add OPENAI_API_KEY, LANGSMITH_API_KEY, etc.
```

### Running Agents as Services
```bash
# Start LangGraph development server with hot-reloading
langgraph dev

# This provides:
# - Local API server (typically http://localhost:8123)
# - LangGraph Studio UI for testing and debugging
# - Hot-reloading during development
```

The `langgraph.json` file defines available agent graphs that can be served.

### Running Notebooks
```bash
# Start Jupyter notebook server
jupyter notebook

# Navigate to notebooks/LG101/ or notebooks/LG201/
```

## Project Structure

### Core Directories

- **`agents/`**: Standalone agent implementations designed to run as services via `langgraph dev`
  - Each agent exports a graph or agent object referenced in `langgraph.json`
  - Agents can be composed (subagents called by supervisors)

- **`notebooks/`**: Educational Jupyter notebooks with tutorials
  - **`LG101/`**: Fundamentals (langgraph_101.ipynb, langgraph_102.ipynb)
  - **`LG201/`**: Advanced patterns (email_agent.ipynb, multi_agent.ipynb)
  - Contains helper files: `models.py`, `utils.py`

### Key Configuration Files

- **`langgraph.json`**: Defines agent graphs available through the LangGraph development server
  - Maps graph names to Python module paths (e.g., `"./agents/email_agent.py:graph"`)
  - Specifies environment file (`.env`) and Python version requirements

- **`pyproject.toml`**: Dependency management using uv/pip
  - Requires Python >=3.13
  - Core dependencies: langchain>=1.0.1, langgraph>=1.0.1, langchain-openai>=1.0.1

## Architecture Patterns

### Agent Architecture

This repository demonstrates two main agent architectures:

1. **Single-Purpose Agents** (e.g., `agents/email_agent.py`)
   - State management using `StateGraph` and `MessagesState`
   - Router pattern for decision-making (triage_router classifies emails)
   - Action agent with tool calling (`llm_call` → `tool_node` cycle)
   - Conditional routing based on tool calls (continues until `Done` tool called)

2. **Multi-Agent Supervisors** (e.g., `agents/music_store_supervisor.py`)
   - Supervisor agent created with `create_agent()` from langchain.agents
   - Tools that wrap subagent graphs (`call_invoice_information_subagent`, `call_music_catalog_subagent`)
   - Uses `ToolRuntime` to access parent state (e.g., `customer_id`)
   - Subagents are invoked as black-box tools and return results to supervisor

### LangGraph v1 Patterns

The codebase uses modern LangGraph v1 patterns:

- **`create_agent()`**: High-level agent creation API (replaces older patterns)
- **State schemas**: TypedDict-based state with annotations (e.g., `Annotated[list[AnyMessage], add_messages]`)
- **`Command` objects**: For conditional routing with state updates
- **`ToolRuntime`**: Access parent agent state within tool implementations
- **Middleware**: Advanced control flow (covered in langgraph_102.ipynb)
- **Human-in-the-loop**: Interrupt patterns for verification (supervisor_with_interrupt.py)

### State Management

Agents use typed state classes inheriting from `TypedDict` or `MessagesState`:
- **`InputState`**: Defines input schema
- **`State`**: Extends InputState with additional fields (customer_id, loaded_memory, etc.)
- Message reducer: `add_messages` annotation for message list merging

### Tool Patterns

Tools in this codebase follow these patterns:
- Decorated with `@tool` or `@tool(name_or_callable=..., description=...)`
- Can accept `ToolRuntime` parameter to access parent state
- Subagent tools invoke compiled graphs and extract response from messages
- `Done` tool pattern to signal completion in agentic loops

## Model Configuration

### Primary LLM Configuration

Model configuration is centralized in `agents/utils.py`:
```python
llm = ChatOpenAI(model_name="gpt-4o", temperature=0)
```

Individual agents may override with specific models (e.g., `music_store_supervisor.py` uses `anthropic:claude-3-7-sonnet-latest`).

### Azure OpenAI Support

To use Azure OpenAI instead of OpenAI:
1. Set environment variables in `.env`: `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_VERSION`
2. Uncomment Azure model configurations in `notebooks/models.py`
3. Update `notebooks/utils.py` to use `AzureOpenAIEmbeddings`
4. Use AzureOpenAI model initialization in notebook cells

## Database and Tools

- **Chinook Database**: In-memory SQLite database loaded from remote SQL script
  - Used by music and invoice agents for catalog/purchase queries
  - Helper function: `get_engine_for_chinook_db()` in `agents/utils.py`

- **Tool Examples**: See `agents/email_agent.py` for tool patterns:
  - `schedule_meeting`, `check_calendar_availability`, `write_email`
  - Placeholder implementations (real apps would integrate with APIs)

## Learning Path

1. Start with `notebooks/LG101/langgraph_101.ipynb` - fundamentals of models, tools, memory, streaming
2. Continue with `notebooks/LG101/langgraph_102.ipynb` - middleware and human-in-the-loop
3. Explore `notebooks/LG201/email_agent.ipynb` - complete stateful agent example
4. Study `notebooks/LG201/multi_agent.ipynb` - multi-agent systems with supervisors
5. Examine `agents/` directory for production-ready implementations
6. Run agents locally with `langgraph dev` to test via API/Studio

## Important Notes

- All code uses **LangChain v1** and **LangGraph v1** APIs (as of January 2025)
- Agents are designed for educational purposes with placeholder tool implementations
- Memory patterns demonstrated in `memory_enabled_music_store_supervisor_with_interrupt.py`
- Interrupt patterns for human verification in `*_with_interrupt.py` agents
