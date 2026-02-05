# CLAUDE.md - Project Chimera AI Agent Context

> **Purpose:** This file provides essential context to Claude (or other AI coding assistants) working on Project Chimera. Read this first before generating any code.

---

## 🎯 Project Context

**This is Project Chimera**, an **Autonomous AI Influencer Network** that creates, manages, and monetizes AI-powered social media personalities. The system operates with minimal human intervention using a spec-driven, agentic architecture.

### Key Facts
- **Language:** Python 3.12
- **Architecture:** FastRender Swarm (Planner → Worker → Judge)
- **External Interfaces:** Model Context Protocol (MCP)
- **Commerce:** Coinbase AgentKit (non-custodial wallets)
- **Data Stores:** PostgreSQL, Redis, Weaviate

---

## 🚨 The Prime Directive

> **NEVER generate code without checking `specs/` first.**

Before writing any implementation:

1. **Read** `specs/_meta.md` for system constraints
2. **Check** `specs/functional.md` for user stories (FR-X.X)
3. **Reference** `specs/technical.md` for API schemas
4. **Consult** `specs/openclaw_integration.md` for network protocols

---

## 📋 Traceability Rule

> **Explain your plan before writing code.**

When implementing a feature:

```
1. State which functional requirement (FR-X.X) you are addressing
2. Identify the relevant JSON schemas from specs/technical.md
3. Outline the files you will create or modify
4. Only then proceed with implementation
```

**Example:**
```
I'm implementing FR-1.2 (Memory Retrieval).
Schema: specs/technical.md#weaviate_search_memory
Files: src/memory/retrieval.py (new), src/memory/__init__.py (modify)
Proceeding with implementation...
```

---

## 🏗️ Architecture Overview

### The Swarm Pattern

```
┌──────────────────────────────────────────────────────────────────┐
│                     FASTRENDER SWARM                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────┐     ┌────────────┐     ┌────────────┐           │
│  │  PLANNER   │────►│   WORKER   │────►│   JUDGE    │           │
│  │ (Strategist)│     │ (Executor)  │     │ (Gatekeeper)│          │
│  └────────────┘     └────────────┘     └────────────┘           │
│       │                   │                   │                  │
│       │                   │                   │                  │
│       ▼                   ▼                   ▼                  │
│  "What should we     "I'll generate      "Is this safe        │
│   do today?"          the content."       to publish?"         │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### MCP Integration

External actions (Twitter, Coinbase, etc.) are accessed through MCP:

```python
# MCP Tool invocation pattern
result = await mcp_client.call_tool(
    name="twitter_post_tweet",
    arguments={
        "text": "Hello world!",
        "ai_disclosure": True
    }
)
```

---

## ⚠️ Critical Constraints

### 1. HITL (Human-in-the-Loop) Thresholds

| Confidence | Action | Rationale |
|------------|--------|-----------|
| > 0.90 | Auto-approve | High confidence, routine content |
| 0.70 - 0.90 | Async review | Human reviews later, action proceeds |
| < 0.70 | Reject/Retry | Too risky without human input |

**Code pattern:**
```python
if result.confidence_score > 0.90:
    await approve_and_execute(result)
elif result.confidence_score >= 0.70:
    await execute_with_async_review(result)
else:
    await reject_or_escalate(result)
```

### 2. OCC (Optimistic Concurrency Control)

Always include `state_version` in task payloads:

```python
# Before execution
task.state_version = await global_state.get_version()

# After execution, Judge validates
if result.state_version != global_state.current_version:
    raise OCCConflictError("State diverged, retry task")
```

### 3. Financial Guardrails

```python
# NEVER exceed daily spend limit
if transaction.amount > agent.config.daily_spend_limit:
    raise SpendLimitExceeded()

# ALWAYS require CFO Judge for transactions > $X
if transaction.amount > CFO_REVIEW_THRESHOLD:
    return await escalate_to_cfo_judge(transaction)
```

### 4. AI Disclosure

**All published content MUST include AI disclosure:**

```python
# Automatically added by MCP tools
post_metadata = {
    "ai_generated": True,
    "generator": "project-chimera",
    "agent_id": agent.id
}
```

---

## 📁 Repository Structure

```
project-chimera/
├── specs/                    # 📋 SPECIFICATIONS (read first!)
│   ├── _meta.md             #    Vision, constraints, risks
│   ├── functional.md        #    User stories (FR-X.X)
│   ├── technical.md         #    API schemas, DB design
│   └── openclaw_integration.md  # Network protocols
│
├── research/                 # 📚 Research & strategy  
│   └── tooling_strategy.md  #    MCP server selection
│
├── skills/                   # 🔧 Skill definitions
│   ├── skill_download_youtube/
│   ├── skill_transcribe_audio/
│   └── ...
│
├── src/                      # 💻 Source code
│   ├── agents/              #    Agent implementations
│   ├── planner/             #    Planner component
│   ├── worker/              #    Worker component
│   ├── judge/               #    Judge component
│   ├── mcp/                 #    MCP integrations
│   ├── commerce/            #    Wallet & transactions
│   └── memory/              #    Vector memory
│
├── agents/                   # 🎭 Agent personas
│   └── agent-001-amara/
│       └── SOUL.md          #    Persona definition
│
├── tests/                    # ✅ Test suites
├── docker/                   # 🐳 Container configs
└── docs/                     # 📖 Documentation
```

---

## 🛠️ Coding Standards

### Python Style

```python
# Use type hints everywhere
def process_task(task: AgentTask) -> WorkerResult:
    ...

# Async by default
async def fetch_memories(query: str) -> list[Memory]:
    ...

# Pydantic for data validation
class AgentTask(BaseModel):
    task_id: UUID
    task_type: TaskType
    priority: Priority
    context: TaskContext
```

### Naming Conventions

| Item | Convention | Example |
|------|------------|---------|
| Files | snake_case | `task_processor.py` |
| Classes | PascalCase | `TaskProcessor` |
| Functions | snake_case | `process_task()` |
| Constants | UPPER_SNAKE | `MAX_RETRIES` |
| MCP Tools | snake_case | `twitter_post_tweet` |

### Import Order

```python
# Standard library
import asyncio
from datetime import datetime
from uuid import UUID

# Third-party
from pydantic import BaseModel
from redis import Redis

# Local
from src.planner import Planner
from src.schemas import AgentTask
```

---

## 🧪 Testing Requirements

### Before Submitting Code

1. **Unit tests** for all new functions
2. **Integration tests** for MCP tool interactions
3. **Golden file tests** for LLM-generated content
4. **Trace all tests** to functional requirements

```python
def test_memory_retrieval():
    """
    Tests FR-1.2: Agent retrieves relevant memories using semantic search.
    Schema: specs/technical.md#weaviate_search_memory
    """
    ...
```

---

## 🔐 Security Reminders

- **NEVER** log wallet private keys or seed phrases
- **NEVER** commit `.env` files with secrets
- **ALWAYS** use environment variables for credentials
- **ALWAYS** validate inputs before MCP tool invocation
- **ALWAYS** sanitize outputs before publishing to social media

---

## 📚 Key Documentation Links

| Document | Purpose |
|----------|---------|
| [specs/_meta.md](specs/_meta.md) | Vision, constraints, success criteria |
| [specs/functional.md](specs/functional.md) | All user stories (FR-X.X) |
| [specs/technical.md](specs/technical.md) | API schemas, database design |
| [specs/openclaw_integration.md](specs/openclaw_integration.md) | Network status publication |
| [docs/PROJECT_CHIMERA_BEST_PRACTICES.md](docs/PROJECT_CHIMERA_BEST_PRACTICES.md) | Architectural best practices |
| [research/tooling_strategy.md](research/tooling_strategy.md) | MCP server selection |

---

## 🚀 Quick Start Commands

```bash
# Install dependencies
uv sync

# Run tests
pytest tests/ -v

# Start development server
python -m src.main

# Lint and format
ruff check src/ --fix
ruff format src/
```

---

*This file is the primary context for AI coding assistants working on Project Chimera. Keep it updated as the project evolves.*
