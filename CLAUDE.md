# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HelloAgents is a lightweight, educational multi-agent framework built on OpenAI's native API. The core design philosophy: **everything except Agent is a Tool** (Memory, RAG, RL, MCP protocols are all abstracted as tools).

## Build and Development Commands

```bash
# Install with all features
pip install -e .[all]

# Install specific features
pip install -e .[search]      # Web search
pip install -e .[memory]      # Memory system (Qdrant, Neo4j)
pip install -e .[rag]         # RAG document Q&A
pip install -e .[protocols]   # MCP/A2A/ANP protocols
pip install -e .[evaluation]  # Benchmarks (GAIA, BFCL)
pip install -e .[rl]          # Reinforcement learning

# Run tests
pytest tests/ -v --tb=short

# Run a single test
pytest tests/test_file.py::test_function -v

# Run examples
python examples/chapter07_basic_setup.py
```

## Architecture

### Core Components (`hello_agents/core/`)
- `llm.py`: `HelloAgentsLLM` - unified LLM client supporting OpenAI, DeepSeek, Qwen, ModelScope, Kimi, Zhipu, Ollama, vLLM with auto-detection
- `agent.py`: Abstract `Agent` base class with `run()` method and message history
- `message.py`: `Message` data class for conversation history

### Agent Paradigms (`hello_agents/agents/`)
- `SimpleAgent`: Basic conversational agent
- `ReActAgent`: Reasoning + Acting loop with tool calls (Thought → Action → Observation)
- `ReflectionAgent`: Self-reflection and iterative improvement
- `PlanAndSolveAgent`: Plan decomposition then step-by-step execution
- `FunctionCallAgent`: OpenAI native function calling
- `ToolAwareSimpleAgent`: SimpleAgent with tool awareness

### Tool System (`hello_agents/tools/`)
- `base.py`: `Tool` abstract base class with `@tool_action` decorator for expandable tools
- `registry.py`: `ToolRegistry` for tool management; supports Tool objects or raw functions
- Built-in tools: SearchTool, CalculatorTool, MemoryTool, RAGTool, MCPWrapperTool, etc.

### Memory System (`hello_agents/memory/`)
Four-layer architecture:
- **Working Memory**: Short-term context
- **Episodic Memory**: Event sequences with temporal relationships
- **Semantic Memory**: Knowledge graphs (Neo4j integration)
- **Perceptual Memory**: Multimodal data processing

Storage backends: Qdrant (vectors), Neo4j (graphs), SQLite (documents)

### Protocols (`hello_agents/protocols/`)
- **MCP**: Model Context Protocol (requires `fastmcp`)
- **A2A**: Agent-to-Agent communication (A2AServer, A2AClient, AgentNetwork)
- **ANP**: Agent Network Protocol for service discovery

### Evaluation (`hello_agents/evaluation/benchmarks/`)
- GAIA benchmark integration
- BFCL (Berkeley Function Calling Leaderboard) - requires separate venv due to numpy version conflict

## Key Patterns

### Creating an Agent with Tools
```python
from hello_agents import ReActAgent, HelloAgentsLLM, ToolRegistry, SearchTool

llm = HelloAgentsLLM()  # Auto-detects provider from env
registry = ToolRegistry()
registry.register_tool(SearchTool())

agent = ReActAgent(name="Assistant", llm=llm, tool_registry=registry)
result = agent.run("Your question here")
```

### Creating Custom Tools
```python
from hello_agents.tools.base import Tool, ToolParameter, tool_action

class MyTool(Tool):
    def __init__(self):
        super().__init__(name="my_tool", description="Does something", expandable=True)

    @tool_action("my_action", "Performs an action")
    def _my_action(self, input_text: str) -> str:
        return f"Result: {input_text}"

    def run(self, parameters): ...
    def get_parameters(self): ...
```

## Environment Configuration

Create `.env` file with:
```bash
LLM_MODEL_ID=your-model-name
LLM_API_KEY=your-api-key
LLM_BASE_URL=your-api-base-url
LLM_TIMEOUT=60

# Optional: Search tools
TAVILY_API_KEY=your-key
```

The framework auto-detects provider from API key format (`ms-` → ModelScope, `sk-` → OpenAI-compatible) or base URL domain.

## Code Style

- Python 3.10+ required
- Uses `black` for formatting (line-length=88)
- Uses `isort` with black profile
- Type hints with `mypy` strict mode
- Tests in `tests/` directory with pytest
