# Project Chimera: Best Practices & Architectural Guidelines

> **Version:** 1.0.0  
> **Last Updated:** February 4, 2026  
> **Scope:** Autonomous Influencer Network Infrastructure

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Spec-Driven Development (SDD)](#2-spec-driven-development-sdd)
3. [Model Context Protocol (MCP) Best Practices](#3-model-context-protocol-mcp-best-practices)
4. [FastRender Swarm Architecture Patterns](#4-fastrender-swarm-architecture-patterns)
5. [Human-in-the-Loop (HITL) Design Patterns](#5-human-in-the-loop-hitl-design-patterns)
6. [Agentic Commerce & Wallet Security](#6-agentic-commerce--wallet-security)
7. [MLOps & CI/CD Pipeline Standards](#7-mlops--cicd-pipeline-standards)
8. [Testing Strategies for AI Agents](#8-testing-strategies-for-ai-agents)
9. [Version Control & Git Hygiene](#9-version-control--git-hygiene)
10. [Monitoring & Observability](#10-monitoring--observability)
11. [Security & Compliance](#11-security--compliance)
12. [Project Structure Standards](#12-project-structure-standards)

---

## 1. Introduction

Project Chimera requires a robust engineering foundation that enables **AI agents to collaborate on the codebase** with minimal human conflict. This document establishes the architectural patterns, coding standards, and operational procedures that govern the development of the Autonomous Influencer Network.

### 1.1 Core Philosophy

```
┌─────────────────────────────────────────────────────────────────┐
│                    THE CHIMERA TRIANGLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        SPECIFICATION                            │
│                            ▲                                    │
│                           / \                                   │
│                          /   \                                  │
│                         /     \                                 │
│                        /       \                                │
│                       /  TRUTH  \                               │
│                      /           \                              │
│                     /             \                             │
│                    ▼───────────────▼                            │
│              INFRASTRUCTURE ◄──► VALIDATION                     │
│                                                                 │
│  • Spec = Source of Truth (What we build)                       │
│  • Infrastructure = CI/CD, Docker, Tests (How we build)         │
│  • Validation = HITL, Judges, Tests (Proof it works)            │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Key Principles

| Principle            | Description                                                |
| -------------------- | ---------------------------------------------------------- |
| **Simplicity First** | Start simple, add complexity only when demonstrably needed |
| **Transparency**     | Show agent planning steps explicitly                       |
| **Traceability**     | Every action must be auditable                             |
| **Loose Coupling**   | Components must be independently deployable                |
| **Fail-Safe Design** | Graceful degradation over catastrophic failure             |

---

## 2. Spec-Driven Development (SDD)

### 2.1 The Golden Rule

> **"We do NOT write implementation code until the Specification is ratified."**

Ambiguity is the enemy of AI agents. Vague specs lead to hallucinations, incorrect implementations, and technical debt.

### 2.2 Specification File Structure

```
specs/
├── AGENTS.md              # Fleet-wide governance & ethical boundaries
├── schemas/
│   ├── task.schema.json   # Agent Task payload schema
│   ├── result.schema.json # Worker Result schema
│   └── persona.schema.json# SOUL.md persona schema
├── features/
│   ├── FR-1.0-persona.md  # Cognitive Core specs
│   ├── FR-2.0-perception.md
│   ├── FR-3.0-creative.md
│   ├── FR-4.0-action.md
│   ├── FR-5.0-commerce.md
│   └── FR-6.0-orchestration.md
└── nfr/
    ├── NFR-1.0-hitl.md    # Non-functional requirements
    ├── NFR-2.0-ethics.md
    └── NFR-3.0-performance.md
```

### 2.3 Specification Template

Every feature spec MUST follow this template:

```markdown
# FR-X.X: [Feature Name]

## Status

- [ ] Draft
- [ ] Review
- [ ] Ratified
- [ ] Implemented

## Overview

[One paragraph describing the feature]

## Acceptance Criteria

- [ ] AC-1: [Specific, measurable criterion]
- [ ] AC-2: [...]

## Dependencies

- Depends on: [FR-X.X, FR-Y.Y]
- Blocks: [FR-Z.Z]

## Technical Constraints

[Any technical limitations or requirements]

## Test Cases

| ID   | Input | Expected Output | Priority |
| ---- | ----- | --------------- | -------- |
| TC-1 | ...   | ...             | P1       |

## AI-Assist Prompt
```

[Genesis prompt for AI coding assistants]

```

## Changelog
| Date | Author | Change |
|------|--------|--------|
```

### 2.4 Schema-First Design

All data contracts MUST be defined in JSON Schema before implementation:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/task.schema.json",
  "title": "Agent Task",
  "type": "object",
  "required": ["task_id", "task_type", "priority", "context"],
  "properties": {
    "task_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique task identifier"
    },
    "task_type": {
      "type": "string",
      "enum": ["generate_content", "reply_comment", "execute_transaction"]
    },
    "priority": {
      "type": "string",
      "enum": ["high", "medium", "low"]
    },
    "context": {
      "type": "object",
      "required": ["goal_description"],
      "properties": {
        "goal_description": { "type": "string" },
        "persona_constraints": {
          "type": "array",
          "items": { "type": "string" }
        },
        "required_resources": {
          "type": "array",
          "items": { "type": "string", "format": "uri" }
        }
      }
    }
  }
}
```

---

## 3. Model Context Protocol (MCP) Best Practices

### 3.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      MCP HOST (Orchestrator)                    │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  MCP Client  │  │  MCP Client  │  │  MCP Client  │          │
│  │  (Twitter)   │  │  (Weaviate)  │  │  (Coinbase)  │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
└─────────┼─────────────────┼─────────────────┼───────────────────┘
          │ STDIO/SSE       │ STDIO/SSE       │ SSE (Remote)
          ▼                 ▼                 ▼
┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ mcp-server-      │ │ mcp-server-      │ │ mcp-server-      │
│ twitter          │ │ weaviate         │ │ coinbase         │
├──────────────────┤ ├──────────────────┤ ├──────────────────┤
│ Tools:           │ │ Tools:           │ │ Tools:           │
│ • post_tweet     │ │ • search_memory  │ │ • transfer       │
│ • reply_tweet    │ │ • store_memory   │ │ • get_balance    │
│ • get_mentions   │ │ • delete_memory  │ │ • deploy_token   │
├──────────────────┤ ├──────────────────┤ ├──────────────────┤
│ Resources:       │ │ Resources:       │ │ Resources:       │
│ • mentions       │ │ • memories       │ │ • wallet_info    │
│ • profile        │ │ • personas       │ │ • transactions   │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

### 3.2 MCP Primitives Usage Guide

| Primitive     | Purpose              | When to Use                               |
| ------------- | -------------------- | ----------------------------------------- |
| **Resources** | Passive data reading | Perception system, polling external state |
| **Tools**     | Active execution     | Actions, mutations, API calls             |
| **Prompts**   | Reusable templates   | Standardized reasoning patterns           |

### 3.3 MCP Server Design Principles

#### 3.3.1 Single Responsibility

Each MCP server should wrap ONE external system:

```python
# ✅ GOOD: One server per domain
mcp-server-twitter    # Only Twitter API
mcp-server-instagram  # Only Instagram API
mcp-server-weaviate   # Only Vector DB

# ❌ BAD: Monolithic server
mcp-server-social     # Multiple platforms mixed
```

#### 3.3.2 Tool Naming Convention

```python
# Pattern: <domain>_<action>_<target>
# Examples:
twitter_post_tweet
twitter_reply_to_mention
weaviate_search_memories
coinbase_transfer_usdc
```

#### 3.3.3 Error Handling

```python
from mcp.types import TextContent, ErrorContent

async def handle_tool_call(name: str, arguments: dict) -> list[TextContent | ErrorContent]:
    try:
        result = await execute_tool(name, arguments)
        return [TextContent(type="text", text=json.dumps(result))]
    except RateLimitError as e:
        return [ErrorContent(
            type="error",
            error_code="RATE_LIMITED",
            message=f"Rate limited. Retry after {e.retry_after}s"
        )]
    except AuthenticationError:
        return [ErrorContent(
            type="error",
            error_code="AUTH_FAILED",
            message="Authentication failed. Check credentials."
        )]
```

#### 3.3.4 Resource URI Schemes

```
# Scheme: <protocol>://<domain>/<path>

# Social Media
twitter://mentions/recent
twitter://user/{user_id}/profile
instagram://feed/latest

# Memory
memory://agent/{agent_id}/recent
memory://agent/{agent_id}/search?q={query}

# Commerce
wallet://balance/usdc
wallet://transactions/recent

# News & Trends
news://ethiopia/fashion/trends
news://global/crypto/latest
```

### 3.4 Transport Selection

| Transport | Use Case                     | Latency | Security            |
| --------- | ---------------------------- | ------- | ------------------- |
| **STDIO** | Local servers, same machine  | Lowest  | Process isolation   |
| **SSE**   | Remote servers, cloud-hosted | Higher  | HTTPS + Auth tokens |

```python
# STDIO Transport (Local)
async with stdio_client(server_config) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()

# SSE Transport (Remote)
async with sse_client(
    url="https://mcp.aiqem.tech/coinbase",
    headers={"Authorization": f"Bearer {token}"}
) as (read, write):
    async with ClientSession(read, write) as session:
        await session.initialize()
```

---

## 4. FastRender Swarm Architecture Patterns

### 4.1 Role Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                     PLANNER (The Strategist)                    │
│  • Reads GlobalState (goals, budget, trends)                    │
│  • Decomposes goals into Task DAG                               │
│  • Dynamic re-planning on context shifts                        │
│  • Can spawn Sub-Planners for domain specialization             │
├─────────────────────────────────────────────────────────────────┤
                              │
                              │ TaskQueue (Redis)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      WORKERS (The Executors)                    │
│  • Stateless, ephemeral agents                                  │
│  • One task = One worker (share-nothing)                        │
│  • Primary MCP Tool consumers                                   │
│  • No peer-to-peer communication                                │
├─────────────────────────────────────────────────────────────────┤
                              │
                              │ ReviewQueue (Redis)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       JUDGE (The Gatekeeper)                    │
│  • Reviews every Worker output                                  │
│  • Validates against persona & safety constraints               │
│  • Implements OCC for state consistency                         │
│  • Authority: APPROVE / REJECT / ESCALATE                       │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Task State Machine

```
                    ┌──────────┐
                    │ PENDING  │
                    └────┬─────┘
                         │ Worker picks up
                         ▼
                  ┌─────────────┐
                  │ IN_PROGRESS │
                  └──────┬──────┘
                         │ Worker completes
                         ▼
                    ┌─────────┐
           ┌────────│ REVIEW  │────────┐
           │        └─────────┘        │
           │              │            │
     REJECTED        APPROVED      ESCALATED
           │              │            │
           ▼              ▼            ▼
    ┌──────────┐   ┌──────────┐  ┌──────────┐
    │ RETRY    │   │ COMPLETE │  │ HITL     │
    └──────────┘   └──────────┘  └──────────┘
```

### 4.3 Optimistic Concurrency Control (OCC)

```python
from dataclasses import dataclass
from typing import Any
import hashlib
import json

@dataclass
class GlobalState:
    version: str  # Hash of state
    campaign_goals: list[str]
    budget_remaining: float
    active_trends: list[str]

    def compute_version(self) -> str:
        state_str = json.dumps({
            "goals": self.campaign_goals,
            "budget": self.budget_remaining,
            "trends": self.active_trends
        }, sort_keys=True)
        return hashlib.sha256(state_str.encode()).hexdigest()[:16]


class JudgeService:
    async def commit_result(
        self,
        result: WorkerResult,
        expected_version: str
    ) -> CommitOutcome:
        """
        OCC Implementation:
        1. Worker starts with state_version = X
        2. Worker completes task
        3. Judge checks: current_version == X?
        4. If YES: commit and increment version
        5. If NO: reject, task is stale
        """
        current_state = await self.get_global_state()

        if current_state.version != expected_version:
            # State drifted during execution
            return CommitOutcome(
                success=False,
                reason="STATE_DRIFT",
                message=f"State changed from {expected_version} to {current_state.version}"
            )

        # Validate result against constraints
        validation = await self.validate_result(result)
        if not validation.passed:
            return CommitOutcome(
                success=False,
                reason="VALIDATION_FAILED",
                message=validation.errors
            )

        # Commit and update version
        await self.apply_result(result)
        new_state = await self.get_global_state()
        new_state.version = new_state.compute_version()
        await self.save_global_state(new_state)

        return CommitOutcome(success=True)
```

### 4.4 Worker Best Practices

```python
class Worker:
    """
    Worker Design Principles:
    1. Stateless - No persistent memory between tasks
    2. Atomic - One task, one execution
    3. Isolated - No peer communication
    4. Focused - Use appropriate LLM tier
    """

    async def execute(self, task: Task) -> WorkerResult:
        # 1. Capture state version at start
        state_version = await self.get_state_version()

        # 2. Load required context via MCP Resources
        context = await self.load_context(task.context.required_resources)

        # 3. Execute using appropriate model tier
        model = self.select_model(task.priority)

        # 4. Perform the work
        try:
            output = await self.llm_execute(model, task, context)
        except Exception as e:
            return WorkerResult(
                task_id=task.task_id,
                status="failed",
                error=str(e),
                state_version=state_version
            )

        # 5. Return result with version for OCC
        return WorkerResult(
            task_id=task.task_id,
            status="pending_review",
            output=output,
            state_version=state_version,
            confidence_score=output.confidence
        )

    def select_model(self, priority: str) -> str:
        """Tiered model selection for cost optimization"""
        return {
            "high": "gemini-3-pro",      # Complex reasoning
            "medium": "gemini-3-flash",   # Balanced
            "low": "gemini-3-flash"       # High volume, simple tasks
        }.get(priority, "gemini-3-flash")
```

---

## 5. Human-in-the-Loop (HITL) Design Patterns

### 5.1 Confidence-Based Routing

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONFIDENCE THRESHOLDS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  HIGH (> 0.90)      AUTO-APPROVE                                │
│  ───────────────────────────────────────────────────────────►   │
│                     Execute immediately, no human needed        │
│                                                                 │
│  MEDIUM (0.70-0.90) ASYNC APPROVAL                              │
│  ───────────────────────────────────────────────────────────►   │
│                     Queue for human review, continue other work │
│                                                                 │
│  LOW (< 0.70)       REJECT/RETRY                                │
│  ───────────────────────────────────────────────────────────►   │
│                     Automatic rejection, Planner re-evaluates   │
│                                                                 │
│  SENSITIVE          MANDATORY REVIEW                            │
│  ───────────────────────────────────────────────────────────►   │
│                     Always human review (politics, health, etc) │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 Sensitive Topic Detection

```python
SENSITIVE_TOPICS = {
    "politics": ["election", "vote", "president", "party", "government"],
    "health": ["diagnosis", "treatment", "medicine", "disease", "cure"],
    "finance": ["invest", "stock", "crypto", "guaranteed returns"],
    "legal": ["lawsuit", "legal advice", "court", "attorney"],
}

class SensitivityFilter:
    def __init__(self):
        self.classifier = load_semantic_classifier()

    async def check(self, content: str) -> SensitivityResult:
        # 1. Keyword matching (fast path)
        keyword_hits = self._keyword_scan(content)

        # 2. Semantic classification (if keyword hits)
        if keyword_hits:
            semantic_score = await self.classifier.classify(content)
            return SensitivityResult(
                is_sensitive=semantic_score > 0.5,
                topics=keyword_hits,
                confidence=semantic_score
            )

        return SensitivityResult(is_sensitive=False)

    def _keyword_scan(self, content: str) -> list[str]:
        content_lower = content.lower()
        hits = []
        for topic, keywords in SENSITIVE_TOPICS.items():
            if any(kw in content_lower for kw in keywords):
                hits.append(topic)
        return hits
```

### 5.3 HITL Queue Implementation

```python
from enum import Enum
from datetime import datetime, timedelta

class ReviewPriority(Enum):
    CRITICAL = 1   # Review within 5 minutes
    HIGH = 2       # Review within 1 hour
    MEDIUM = 3     # Review within 4 hours
    LOW = 4        # Review within 24 hours

@dataclass
class HITLTask:
    task_id: str
    content: str
    content_type: str  # "text" | "image" | "transaction"
    confidence_score: float
    sensitivity_flags: list[str]
    priority: ReviewPriority
    created_at: datetime
    expires_at: datetime
    agent_id: str
    reasoning_trace: str  # Why the agent made this decision

class HITLQueue:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.queue_key = "hitl:review_queue"

    async def enqueue(self, task: HITLTask) -> None:
        # Priority queue using Redis sorted sets
        score = task.priority.value * 1e12 + task.created_at.timestamp()
        await self.redis.zadd(
            self.queue_key,
            {task.task_id: score}
        )
        await self.redis.hset(
            f"hitl:task:{task.task_id}",
            mapping=task.to_dict()
        )

    async def get_next(self) -> HITLTask | None:
        # Get highest priority (lowest score) task
        result = await self.redis.zrange(self.queue_key, 0, 0)
        if not result:
            return None

        task_id = result[0]
        task_data = await self.redis.hgetall(f"hitl:task:{task_id}")
        return HITLTask.from_dict(task_data)
```

---

## 6. Agentic Commerce & Wallet Security

### 6.1 Wallet Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    WALLET SECURITY LAYERS                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Layer 1: Key Storage                                           │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  AWS Secrets Manager / HashiCorp Vault                  │    │
│  │  • Private keys encrypted at rest                       │    │
│  │  • Automatic rotation policies                          │    │
│  │  • Audit logging for all access                         │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                  │
│                              ▼                                  │
│  Layer 2: Runtime Injection                                     │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Environment Variables (Container Startup Only)         │    │
│  │  • Keys injected at container start                     │    │
│  │  • Never logged or printed                              │    │
│  │  • Memory-only, not written to disk                     │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                  │
│                              ▼                                  │
│  Layer 3: CFO Judge (Budget Governance)                         │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Transaction Review Agent                               │    │
│  │  • Enforces daily/weekly/monthly limits                 │    │
│  │  • Anomaly detection                                    │    │
│  │  • Whitelist validation                                 │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Budget Governance Implementation

```python
from functools import wraps
from decimal import Decimal
import redis.asyncio as redis

class BudgetExceededError(Exception):
    pass

class CFOJudge:
    """
    The CFO Judge reviews every transaction before execution.
    It enforces configurable budget limits and anomaly detection.
    """

    def __init__(self, redis_client: redis.Redis, config: BudgetConfig):
        self.redis = redis_client
        self.config = config

    async def review_transaction(
        self,
        agent_id: str,
        transaction: TransactionRequest
    ) -> TransactionDecision:
        # 1. Check daily limit
        daily_spend = await self._get_daily_spend(agent_id)
        if daily_spend + transaction.amount > self.config.daily_limit:
            return TransactionDecision(
                approved=False,
                reason="DAILY_LIMIT_EXCEEDED",
                message=f"Daily limit: ${self.config.daily_limit}, "
                        f"Current: ${daily_spend}, Requested: ${transaction.amount}"
            )

        # 2. Check single transaction limit
        if transaction.amount > self.config.single_tx_limit:
            return TransactionDecision(
                approved=False,
                reason="SINGLE_TX_LIMIT_EXCEEDED",
                escalate_to_human=True
            )

        # 3. Check recipient whitelist (if enabled)
        if self.config.require_whitelist:
            if transaction.to_address not in self.config.whitelist:
                return TransactionDecision(
                    approved=False,
                    reason="RECIPIENT_NOT_WHITELISTED",
                    escalate_to_human=True
                )

        # 4. Anomaly detection
        anomaly_score = await self._detect_anomaly(agent_id, transaction)
        if anomaly_score > 0.8:
            return TransactionDecision(
                approved=False,
                reason="ANOMALY_DETECTED",
                anomaly_score=anomaly_score,
                escalate_to_human=True
            )

        return TransactionDecision(approved=True)

    async def _get_daily_spend(self, agent_id: str) -> Decimal:
        key = f"budget:{agent_id}:daily:{datetime.utcnow().date()}"
        value = await self.redis.get(key)
        return Decimal(value) if value else Decimal(0)

    async def record_spend(self, agent_id: str, amount: Decimal) -> None:
        key = f"budget:{agent_id}:daily:{datetime.utcnow().date()}"
        await self.redis.incrbyfloat(key, float(amount))
        await self.redis.expire(key, 86400 * 2)  # Keep for 2 days


def budget_check(func):
    """Decorator to enforce budget checks on financial operations"""
    @wraps(func)
    async def wrapper(self, *args, **kwargs):
        transaction = kwargs.get('transaction') or args[0]

        decision = await self.cfo_judge.review_transaction(
            self.agent_id,
            transaction
        )

        if not decision.approved:
            if decision.escalate_to_human:
                await self.hitl_queue.enqueue(
                    HITLTask(
                        content=f"Transaction review: {transaction}",
                        content_type="transaction",
                        reasoning_trace=decision.reason
                    )
                )
            raise BudgetExceededError(decision.message)

        result = await func(self, *args, **kwargs)

        # Record successful spend
        await self.cfo_judge.record_spend(self.agent_id, transaction.amount)

        return result
    return wrapper
```

### 6.3 AgentKit Integration

```python
from coinbase_agentkit import (
    AgentKit,
    CdpV2WalletProvider,
)
import os

class CommerceManager:
    def __init__(self, agent_id: str):
        self.agent_id = agent_id
        self._validate_environment()
        self.wallet_provider = None
        self.agentkit = None

    def _validate_environment(self) -> None:
        """Fail fast if credentials are missing"""
        required_vars = [
            "CDP_API_KEY_ID",
            "CDP_API_KEY_SECRET",
            "CDP_WALLET_SECRET"
        ]
        missing = [v for v in required_vars if not os.environ.get(v)]
        if missing:
            raise EnvironmentError(
                f"Missing required environment variables: {missing}"
            )

    async def initialize(self) -> None:
        self.wallet_provider = await CdpV2WalletProvider.configure_with_wallet({
            "apiKeyId": os.environ["CDP_API_KEY_ID"],
            "apiKeySecret": os.environ["CDP_API_KEY_SECRET"],
            "walletSecret": os.environ["CDP_WALLET_SECRET"],
            "networkId": os.environ.get("NETWORK_ID", "base-sepolia"),
        })

        self.agentkit = await AgentKit.from({
            "walletProvider": self.wallet_provider,
            "actionProviders": [],  # Add custom providers as needed
        })

    async def get_balance(self, asset: str = "USDC") -> Decimal:
        """Check wallet balance before any operation"""
        return await self.agentkit.get_balance(asset)

    @budget_check
    async def transfer(self, transaction: TransactionRequest) -> TxReceipt:
        """Execute a transfer with budget governance"""
        return await self.agentkit.transfer(
            to=transaction.to_address,
            amount=str(transaction.amount),
            asset=transaction.asset
        )
```

---

## 7. MLOps & CI/CD Pipeline Standards

### 7.1 Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      CI/CD PIPELINE STAGES                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐      │
│  │  CODE   │───►│  TEST   │───►│  BUILD  │───►│ DEPLOY  │      │
│  │ COMMIT  │    │ & LINT  │    │ IMAGES  │    │ STAGING │      │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘      │
│                                                     │           │
│                                                     ▼           │
│                               ┌─────────────────────────────┐   │
│                               │     INTEGRATION TESTS       │   │
│                               │     (Agent Swarm E2E)       │   │
│                               └─────────────┬───────────────┘   │
│                                             │                   │
│                           ┌─────────────────┴─────────────────┐ │
│                           │                                   │ │
│                     ┌─────▼─────┐                   ┌────────▼┐│
│                     │  CANARY   │                   │ ROLLBACK││
│                     │  DEPLOY   │                   │         ││
│                     └─────┬─────┘                   └─────────┘│
│                           │                                    │
│                     ┌─────▼─────┐                              │
│                     │  PROD     │                              │
│                     │  RELEASE  │                              │
│                     └───────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 7.2 GitHub Actions Workflow

```yaml
# .github/workflows/chimera-ci.yml
name: Chimera CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: "3.12"
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint-and-type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: |
          pip install ruff mypy
          pip install -r requirements.txt

      - name: Lint with Ruff
        run: ruff check --output-format=github .

      - name: Type check with mypy
        run: mypy src/ --strict

  unit-tests:
    runs-on: ubuntu-latest
    needs: lint-and-type-check
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install dependencies
        run: pip install -r requirements.txt -r requirements-dev.txt

      - name: Run unit tests
        run: pytest tests/unit/ -v --cov=src --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage.xml

  schema-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate JSON Schemas
        run: |
          pip install check-jsonschema
          for schema in specs/schemas/*.json; do
            check-jsonschema --check-metaschema "$schema"
          done

  integration-tests:
    runs-on: ubuntu-latest
    needs: [unit-tests, schema-validation]
    services:
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
      weaviate:
        image: semitechnologies/weaviate:latest
        ports:
          - 8080:8080
    steps:
      - uses: actions/checkout@v4

      - name: Run integration tests
        run: pytest tests/integration/ -v
        env:
          REDIS_URL: redis://localhost:6379
          WEAVIATE_URL: http://localhost:8080

  build-and-push:
    runs-on: ubuntu-latest
    needs: integration-tests
    if: github.ref == 'refs/heads/main'
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker images
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

### 7.3 Automation Maturity Levels

| Level       | Description                                        | Chimera Target |
| ----------- | -------------------------------------------------- | -------------- |
| **Level 0** | Manual - All steps hand-executed                   | ❌             |
| **Level 1** | ML Pipeline Automation - Auto-training on new data | ✅ Phase 1     |
| **Level 2** | CI/CD Automation - Auto-deploy on passing tests    | ✅ Phase 2     |
| **Level 3** | Full MLOps - Continuous training + monitoring      | ✅ Phase 3     |

---

## 8. Testing Strategies for AI Agents

### 8.1 Testing Pyramid for Agents

```
                         ▲
                        /│\
                       / │ \
                      /  │  \        E2E / Agent Behavior Tests
                     /   │   \       (Slow, Expensive, Essential)
                    /    │    \
                   /─────┼─────\
                  /      │      \    Integration Tests
                 /       │       \   (MCP Servers, Queues)
                /        │        \
               /─────────┼─────────\
              /          │          \  Unit Tests
             /           │           \ (Functions, Classes)
            /            │            \
           /─────────────┼─────────────\
          /              │              \ Schema Validation
         /               │               \ (Data Contracts)
        ▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
```

### 8.2 Test Categories

#### 8.2.1 Schema Validation Tests

```python
import pytest
from jsonschema import validate, ValidationError
import json

class TestSchemas:
    @pytest.fixture
    def task_schema(self):
        with open("specs/schemas/task.schema.json") as f:
            return json.load(f)

    def test_valid_task(self, task_schema):
        valid_task = {
            "task_id": "550e8400-e29b-41d4-a716-446655440000",
            "task_type": "generate_content",
            "priority": "high",
            "context": {
                "goal_description": "Create a fashion post",
                "persona_constraints": ["professional", "trendy"],
                "required_resources": ["mcp://memory/recent"]
            }
        }
        validate(instance=valid_task, schema=task_schema)

    def test_invalid_task_missing_required(self, task_schema):
        invalid_task = {
            "task_id": "550e8400-e29b-41d4-a716-446655440000",
            # Missing task_type, priority, context
        }
        with pytest.raises(ValidationError):
            validate(instance=invalid_task, schema=task_schema)
```

#### 8.2.2 Unit Tests for Agent Components

```python
import pytest
from unittest.mock import AsyncMock, patch

class TestJudgeService:
    @pytest.fixture
    def judge(self):
        return JudgeService(
            redis_client=AsyncMock(),
            config=JudgeConfig(
                high_confidence_threshold=0.90,
                low_confidence_threshold=0.70
            )
        )

    @pytest.mark.asyncio
    async def test_high_confidence_auto_approves(self, judge):
        result = WorkerResult(
            task_id="test-123",
            status="pending_review",
            output={"text": "Hello world"},
            confidence_score=0.95
        )

        decision = await judge.review(result)

        assert decision.action == "APPROVE"
        assert decision.requires_human == False

    @pytest.mark.asyncio
    async def test_low_confidence_rejects(self, judge):
        result = WorkerResult(
            task_id="test-123",
            status="pending_review",
            output={"text": "Maybe hello?"},
            confidence_score=0.60
        )

        decision = await judge.review(result)

        assert decision.action == "REJECT"
        assert decision.retry_recommended == True

    @pytest.mark.asyncio
    async def test_sensitive_content_escalates(self, judge):
        result = WorkerResult(
            task_id="test-123",
            status="pending_review",
            output={"text": "Vote for candidate X!"},
            confidence_score=0.95  # High confidence but sensitive
        )

        decision = await judge.review(result)

        assert decision.action == "ESCALATE"
        assert decision.requires_human == True
        assert "politics" in decision.sensitivity_flags
```

#### 8.2.3 Integration Tests for MCP Servers

```python
import pytest
from mcp import ClientSession
from mcp.client.stdio import stdio_client

class TestMCPTwitterServer:
    @pytest.fixture
    async def twitter_session(self):
        server_config = {
            "command": "python",
            "args": ["-m", "mcp_servers.twitter"]
        }
        async with stdio_client(server_config) as (read, write):
            async with ClientSession(read, write) as session:
                await session.initialize()
                yield session

    @pytest.mark.asyncio
    async def test_list_tools(self, twitter_session):
        tools = await twitter_session.list_tools()

        tool_names = [t.name for t in tools.tools]
        assert "twitter_post_tweet" in tool_names
        assert "twitter_reply_to_mention" in tool_names

    @pytest.mark.asyncio
    async def test_post_tweet_validation(self, twitter_session):
        # Test that tool validates input correctly
        result = await twitter_session.call_tool(
            "twitter_post_tweet",
            {"text": ""}  # Empty text should fail
        )

        assert result.isError
        assert "text cannot be empty" in result.content[0].text.lower()
```

#### 8.2.4 E2E Agent Behavior Tests

```python
class TestAgentBehavior:
    """End-to-end tests for full agent workflows"""

    @pytest.mark.asyncio
    @pytest.mark.slow
    async def test_planner_decomposes_goal(self, agent_runtime):
        goal = "Create a viral fashion post about Ethiopian coffee"

        tasks = await agent_runtime.planner.decompose(goal)

        # Verify task decomposition
        assert len(tasks) >= 3
        task_types = [t.task_type for t in tasks]
        assert "research" in task_types or "generate_content" in task_types

    @pytest.mark.asyncio
    @pytest.mark.slow
    async def test_full_content_generation_flow(self, agent_runtime):
        """Test complete Planner -> Worker -> Judge flow"""
        task = Task(
            task_id="e2e-test-001",
            task_type="generate_content",
            priority="medium",
            context=TaskContext(
                goal_description="Write a short caption about morning coffee",
                persona_constraints=["friendly", "casual"]
            )
        )

        # Execute full flow
        result = await agent_runtime.execute_task(task)

        # Verify completion
        assert result.status in ["complete", "escalated"]
        if result.status == "complete":
            assert len(result.output["text"]) > 0
            assert result.confidence_score >= 0.70
```

### 8.3 ML Test Score Checklist

| Category    | Test                        | Points |
| ----------- | --------------------------- | ------ |
| **Data**    | Schema validation automated | 1.0    |
| **Data**    | Feature importance tracked  | 0.5    |
| **Model**   | Training reproducible       | 1.0    |
| **Model**   | Model staleness monitored   | 0.5    |
| **Infra**   | Integration tests automated | 1.0    |
| **Infra**   | Canary deployments          | 0.5    |
| **Monitor** | Model decay detection       | 1.0    |
| **Monitor** | Alerting on anomalies       | 0.5    |

**Target Score:** > 3.0 (Strong automation level)

---

## 9. Version Control & Git Hygiene

### 9.1 Branching Strategy

```
main (production)
  │
  ├── develop (integration)
  │     │
  │     ├── feature/FR-1.0-persona-system
  │     ├── feature/FR-2.0-perception-engine
  │     └── feature/FR-5.0-agentic-commerce
  │
  └── hotfix/critical-budget-overflow
```

### 9.2 Commit Message Convention

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**

- `feat`: New feature (FR-X.X implementation)
- `fix`: Bug fix
- `spec`: Specification changes
- `refactor`: Code refactoring
- `test`: Test additions/changes
- `docs`: Documentation updates
- `ci`: CI/CD changes

**Examples:**

```
feat(commerce): implement CFO Judge budget governance

- Add daily/weekly/monthly spending limits
- Implement anomaly detection for transactions
- Add whitelist validation for recipients

Closes #42
Implements FR-5.2

---

spec(persona): ratify SOUL.md schema v1.0

- Define required fields: backstory, voice_traits, directives
- Add JSON Schema validation
- Update AI-assist prompts

Ratifies SPEC-FR-1.0
```

### 9.3 Required Files for Git

```
.
├── .github/
│   ├── CODEOWNERS           # Require reviews from code owners
│   ├── pull_request_template.md
│   └── workflows/
│       └── chimera-ci.yml
├── .gitignore
├── .pre-commit-config.yaml  # Pre-commit hooks
├── AGENTS.md                # AI agent governance rules
├── CHANGELOG.md
├── CONTRIBUTING.md
└── README.md
```

### 9.4 Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-json
      - id: check-yaml

  - repo: local
    hooks:
      - id: schema-validation
        name: Validate JSON Schemas
        entry: python scripts/validate_schemas.py
        language: python
        files: ^specs/schemas/.*\.json$
```

---

## 10. Monitoring & Observability

### 10.1 Metrics Categories

```
┌─────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY PILLARS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   METRICS   │  │    LOGS     │  │   TRACES    │             │
│  ├─────────────┤  ├─────────────┤  ├─────────────┤             │
│  │ • Latency   │  │ • Errors    │  │ • Request   │             │
│  │ • Throughput│  │ • Warnings  │  │   flow      │             │
│  │ • Error rate│  │ • Audit     │  │ • Agent     │             │
│  │ • Budget    │  │ • Debug     │  │   decisions │             │
│  └─────────────┘  └─────────────┘  └─────────────┘             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 10.2 Key Metrics to Track

| Metric                      | Description                           | Alert Threshold |
| --------------------------- | ------------------------------------- | --------------- |
| `agent.task.latency_ms`     | Time from task creation to completion | P99 > 10,000ms  |
| `agent.task.success_rate`   | % of tasks completing successfully    | < 95%           |
| `agent.confidence.mean`     | Average confidence score              | < 0.75          |
| `agent.hitl.queue_depth`    | Pending human reviews                 | > 100           |
| `agent.wallet.balance_usdc` | Current wallet balance                | < $10           |
| `agent.wallet.daily_spend`  | Daily transaction volume              | > $limit        |
| `mcp.tool.error_rate`       | MCP tool call failures                | > 5%            |
| `swarm.worker.utilization`  | Worker pool usage                     | > 90% sustained |

### 10.3 Structured Logging

```python
import structlog
from datetime import datetime

logger = structlog.get_logger()

class AgentLogger:
    def __init__(self, agent_id: str):
        self.logger = logger.bind(
            agent_id=agent_id,
            service="chimera-agent"
        )

    def log_task_started(self, task: Task):
        self.logger.info(
            "task_started",
            task_id=task.task_id,
            task_type=task.task_type,
            priority=task.priority
        )

    def log_tool_call(self, tool: str, duration_ms: float, success: bool):
        self.logger.info(
            "mcp_tool_called",
            tool_name=tool,
            duration_ms=duration_ms,
            success=success
        )

    def log_decision(self, decision: JudgeDecision):
        self.logger.info(
            "judge_decision",
            task_id=decision.task_id,
            action=decision.action,
            confidence=decision.confidence_score,
            escalated=decision.requires_human,
            reasoning=decision.reasoning_trace
        )
```

### 10.4 Model Decay Detection

```python
from dataclasses import dataclass
from collections import deque

@dataclass
class ModelMetrics:
    precision: float
    recall: float
    f1_score: float
    timestamp: datetime

class ModelDecayMonitor:
    def __init__(self, window_size: int = 100):
        self.metrics_history = deque(maxlen=window_size)
        self.baseline_f1 = None
        self.decay_threshold = 0.10  # 10% degradation triggers alert

    def record_metrics(self, metrics: ModelMetrics):
        self.metrics_history.append(metrics)

        if self.baseline_f1 is None and len(self.metrics_history) >= 10:
            # Establish baseline from first 10 samples
            self.baseline_f1 = sum(
                m.f1_score for m in list(self.metrics_history)[:10]
            ) / 10

    def check_decay(self) -> DecayAlert | None:
        if self.baseline_f1 is None or len(self.metrics_history) < 20:
            return None

        # Calculate recent average
        recent_f1 = sum(
            m.f1_score for m in list(self.metrics_history)[-10:]
        ) / 10

        degradation = (self.baseline_f1 - recent_f1) / self.baseline_f1

        if degradation > self.decay_threshold:
            return DecayAlert(
                severity="HIGH",
                baseline_f1=self.baseline_f1,
                current_f1=recent_f1,
                degradation_pct=degradation * 100,
                recommendation="Consider retraining model with recent data"
            )

        return None
```

---

## 11. Security & Compliance

### 11.1 Security Checklist

| Category    | Requirement               | Implementation                          |
| ----------- | ------------------------- | --------------------------------------- |
| **Secrets** | No hardcoded credentials  | Environment variables + Secrets Manager |
| **Secrets** | Key rotation              | Automated 90-day rotation               |
| **Auth**    | MCP server authentication | Bearer tokens + OAuth 2.0               |
| **Auth**    | API rate limiting         | Per-agent quotas                        |
| **Audit**   | All actions logged        | Immutable audit trail                   |
| **Audit**   | Transaction records       | On-chain + off-chain logs               |
| **Data**    | Tenant isolation          | Namespace-level separation              |
| **Data**    | PII handling              | Encryption at rest + in transit         |

### 11.2 AI Transparency Requirements (EU AI Act)

```python
class TransparencyManager:
    """Ensures compliance with AI disclosure requirements"""

    DISCLOSURE_PATTERNS = [
        r"are you (?:a |an )?(?:robot|ai|bot|machine|artificial)",
        r"is this (?:a |an )?(?:ai|bot|automated)",
        r"who am i (?:talking|speaking) (?:to|with)",
        r"are you (?:real|human)",
    ]

    def __init__(self, persona: AgentPersona):
        self.persona = persona
        self.compiled_patterns = [
            re.compile(p, re.IGNORECASE) for p in self.DISCLOSURE_PATTERNS
        ]

    def requires_disclosure(self, user_input: str) -> bool:
        """Check if user is asking about AI identity"""
        return any(p.search(user_input) for p in self.compiled_patterns)

    def get_disclosure_response(self) -> str:
        """Generate honest disclosure that respects persona"""
        return (
            f"I'm {self.persona.name}, a virtual persona created by AI. "
            f"I'm here to share content about {self.persona.niche} and engage "
            f"with our community. Is there something specific I can help you with?"
        )

    def add_content_labels(self, content: dict) -> dict:
        """Add AI-generated labels to content payloads"""
        content["ai_disclosure"] = {
            "is_ai_generated": True,
            "generator": "Project Chimera",
            "version": __version__,
            "timestamp": datetime.utcnow().isoformat()
        }
        return content
```

### 11.3 Data Isolation Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     MULTI-TENANT ISOLATION                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Tenant A                          Tenant B                     │
│  ┌────────────────────┐           ┌────────────────────┐       │
│  │ namespace: tenant-a │           │ namespace: tenant-b │       │
│  ├────────────────────┤           ├────────────────────┤       │
│  │ Weaviate Class:    │           │ Weaviate Class:    │       │
│  │ TenantA_Memory     │           │ TenantB_Memory     │       │
│  │                    │           │                    │       │
│  │ Redis Prefix:      │           │ Redis Prefix:      │       │
│  │ tenant-a:*         │           │ tenant-b:*         │       │
│  │                    │           │                    │       │
│  │ Wallet:            │           │ Wallet:            │       │
│  │ 0xAAA...          │           │ 0xBBB...          │       │
│  └────────────────────┘           └────────────────────┘       │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                     ISOLATION BOUNDARY                          │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 12. Project Structure Standards

### 12.1 Recommended Directory Layout

```
project-chimera/
├── .github/
│   ├── CODEOWNERS
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── copilot-instructions.md
├── .vscode/
│   ├── settings.json
│   └── mcp.json
├── docs/
│   ├── PROJECT_CHIMERA_BEST_PRACTICES.md
│   ├── architecture/
│   │   ├── system-overview.md
│   │   └── diagrams/
│   └── api/
│       └── mcp-servers.md
├── specs/
│   ├── AGENTS.md
│   ├── schemas/
│   │   ├── task.schema.json
│   │   ├── result.schema.json
│   │   └── persona.schema.json
│   ├── features/
│   │   ├── FR-1.0-cognitive-core.md
│   │   └── ...
│   └── nfr/
│       └── NFR-1.0-hitl.md
├── src/
│   ├── __init__.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── persona.py
│   │   ├── memory.py
│   │   └── context.py
│   ├── swarm/
│   │   ├── __init__.py
│   │   ├── planner.py
│   │   ├── worker.py
│   │   └── judge.py
│   ├── mcp_servers/
│   │   ├── __init__.py
│   │   ├── twitter/
│   │   ├── weaviate/
│   │   └── coinbase/
│   ├── commerce/
│   │   ├── __init__.py
│   │   ├── wallet.py
│   │   └── cfo_judge.py
│   └── orchestrator/
│       ├── __init__.py
│       └── dashboard.py
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   ├── test_persona.py
│   │   ├── test_judge.py
│   │   └── test_budget.py
│   ├── integration/
│   │   ├── test_mcp_twitter.py
│   │   └── test_swarm_flow.py
│   └── e2e/
│       └── test_agent_behavior.py
├── scripts/
│   ├── validate_schemas.py
│   └── setup_dev.sh
├── docker/
│   ├── Dockerfile.planner
│   ├── Dockerfile.worker
│   ├── Dockerfile.judge
│   └── docker-compose.yml
├── personas/
│   └── SOUL.md.template
├── .env.example
├── .gitignore
├── .pre-commit-config.yaml
├── pyproject.toml
├── requirements.txt
├── requirements-dev.txt
├── AGENTS.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

### 12.2 AGENTS.md Template

```markdown
# AGENTS.md - Project Chimera Governance

## Purpose

This file defines the rules, constraints, and operational boundaries
for all AI agents working within the Project Chimera codebase.

## Code Modification Rules

### Allowed

- Implementing features defined in ratified specifications
- Adding tests for existing functionality
- Refactoring with test coverage
- Documentation updates

### Prohibited

- Modifying AGENTS.md without human approval
- Implementing features without ratified specs
- Removing or weakening security controls
- Adding external dependencies without review

## Commit Guidelines

- All commits must reference a spec (e.g., "Implements FR-1.0")
- Tests must pass before commit
- No `# TODO` without linked issue

## Quality Gates

- [ ] All tests pass
- [ ] Type checking passes (mypy --strict)
- [ ] Lint passes (ruff check)
- [ ] Schema validation passes
- [ ] No secrets in code

## Escalation Triggers

Agents MUST escalate to humans when:

- Confidence score < 0.70
- Sensitive topics detected
- Budget limits approached
- Security-related changes needed
- Spec ambiguity discovered
```

---

## Summary

This document establishes the foundational best practices for Project Chimera. Key takeaways:

1. **Spec-First**: No implementation without ratified specifications
2. **MCP Everywhere**: All external interactions through standardized MCP servers
3. **Swarm Pattern**: Planner → Worker → Judge with OCC for consistency
4. **HITL by Design**: Confidence-based routing with mandatory review for sensitive content
5. **Budget Governance**: CFO Judge reviews all financial transactions
6. **Test Everything**: Schema validation → Unit → Integration → E2E
7. **Observable by Default**: Structured logging, metrics, and traces
8. **Secure by Design**: Tenant isolation, key management, AI transparency

---

**Document Control:**

- **Author:** Project Chimera Architecture Team
- **Review Cycle:** Monthly
- **Next Review:** March 4, 2026
