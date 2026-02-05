# Project Chimera: Technical Specifications

> **Version:** 1.0.0  
> **Status:** Draft  
> **Parent:** [specs/\_meta.md](_meta.md)  
> **Last Updated:** February 5, 2026

---

## 1. Overview

This document defines the technical specifications for Project Chimera, including:

- API Contracts (JSON Input/Output schemas)
- Database Schemas (ERD and table definitions)
- System Interfaces
- Data Flow Diagrams

---

## 2. API Contracts

### 2.1 Agent Task Schema

The standardized payload passed between Planner, Worker, and Judge components.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/task.schema.json",
  "title": "AgentTask",
  "description": "A task assigned by the Planner to a Worker",
  "type": "object",
  "required": [
    "task_id",
    "task_type",
    "priority",
    "context",
    "created_at",
    "status"
  ],
  "properties": {
    "task_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique identifier for the task",
      "examples": ["550e8400-e29b-41d4-a716-446655440000"]
    },
    "task_type": {
      "type": "string",
      "enum": [
        "generate_content",
        "reply_comment",
        "execute_transaction",
        "fetch_trends",
        "analyze_sentiment",
        "validate_image"
      ],
      "description": "The type of task to execute"
    },
    "priority": {
      "type": "string",
      "enum": ["critical", "high", "medium", "low"],
      "description": "Task priority level"
    },
    "context": {
      "type": "object",
      "required": ["goal_description", "agent_id"],
      "properties": {
        "goal_description": {
          "type": "string",
          "description": "Human-readable description of what needs to be done",
          "examples": [
            "Create an Instagram post about Ethiopian coffee culture"
          ]
        },
        "agent_id": {
          "type": "string",
          "description": "The agent this task belongs to"
        },
        "persona_constraints": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Voice/behavior constraints from SOUL.md"
        },
        "required_resources": {
          "type": "array",
          "items": { "type": "string", "format": "uri" },
          "description": "MCP Resource URIs needed for this task",
          "examples": ["mcp://twitter/mentions/123", "mcp://memory/recent"]
        },
        "budget_limit_usd": {
          "type": "number",
          "minimum": 0,
          "description": "Maximum spend allowed for this task"
        },
        "parent_task_id": {
          "type": "string",
          "format": "uuid",
          "description": "Parent task ID if this is a subtask"
        }
      }
    },
    "assigned_worker_id": {
      "type": "string",
      "description": "ID of the Worker assigned to this task"
    },
    "state_version": {
      "type": "string",
      "description": "GlobalState version when task was created (for OCC)"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp of task creation"
    },
    "started_at": {
      "type": "string",
      "format": "date-time",
      "description": "When the Worker started execution"
    },
    "completed_at": {
      "type": "string",
      "format": "date-time",
      "description": "When the task was completed"
    },
    "status": {
      "type": "string",
      "enum": [
        "pending",
        "in_progress",
        "review",
        "approved",
        "rejected",
        "escalated",
        "complete",
        "failed"
      ],
      "description": "Current task status"
    },
    "retry_count": {
      "type": "integer",
      "minimum": 0,
      "default": 0,
      "description": "Number of retry attempts"
    },
    "max_retries": {
      "type": "integer",
      "minimum": 0,
      "default": 3,
      "description": "Maximum retry attempts before failure"
    }
  }
}
```

---

### 2.2 Worker Result Schema

The output payload from a Worker, sent to the Judge for review.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/result.schema.json",
  "title": "WorkerResult",
  "description": "Result of a Worker's task execution",
  "type": "object",
  "required": [
    "result_id",
    "task_id",
    "status",
    "output",
    "confidence_score",
    "state_version"
  ],
  "properties": {
    "result_id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique identifier for this result"
    },
    "task_id": {
      "type": "string",
      "format": "uuid",
      "description": "The task this result belongs to"
    },
    "worker_id": {
      "type": "string",
      "description": "ID of the Worker that produced this result"
    },
    "status": {
      "type": "string",
      "enum": ["success", "partial", "failed"],
      "description": "Execution status"
    },
    "output": {
      "type": "object",
      "description": "The actual output of the task",
      "properties": {
        "content_type": {
          "type": "string",
          "enum": ["text", "image", "video", "transaction", "analysis"],
          "description": "Type of content produced"
        },
        "text": {
          "type": "string",
          "description": "Generated text content (captions, replies)"
        },
        "media_urls": {
          "type": "array",
          "items": { "type": "string", "format": "uri" },
          "description": "URLs to generated media"
        },
        "transaction_hash": {
          "type": "string",
          "description": "Blockchain transaction hash if applicable"
        },
        "metadata": {
          "type": "object",
          "description": "Additional output metadata"
        }
      }
    },
    "confidence_score": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0,
      "description": "Worker's confidence in the output quality"
    },
    "sensitivity_flags": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": ["politics", "health", "finance", "legal", "violence", "adult"]
      },
      "description": "Detected sensitive topic flags"
    },
    "reasoning_trace": {
      "type": "string",
      "description": "Explanation of how the output was generated"
    },
    "state_version": {
      "type": "string",
      "description": "GlobalState version when Worker started (for OCC)"
    },
    "execution_time_ms": {
      "type": "integer",
      "description": "Time taken to execute the task in milliseconds"
    },
    "tools_used": {
      "type": "array",
      "items": { "type": "string" },
      "description": "MCP Tools invoked during execution"
    },
    "error": {
      "type": "object",
      "description": "Error details if status is 'failed'",
      "properties": {
        "code": { "type": "string" },
        "message": { "type": "string" },
        "stack_trace": { "type": "string" }
      }
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

### 2.3 Judge Decision Schema

The output of the Judge's review process.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/decision.schema.json",
  "title": "JudgeDecision",
  "description": "Judge's decision on a Worker result",
  "type": "object",
  "required": ["decision_id", "result_id", "action", "decided_at"],
  "properties": {
    "decision_id": {
      "type": "string",
      "format": "uuid"
    },
    "result_id": {
      "type": "string",
      "format": "uuid",
      "description": "The WorkerResult being judged"
    },
    "task_id": {
      "type": "string",
      "format": "uuid"
    },
    "action": {
      "type": "string",
      "enum": ["approve", "reject", "escalate"],
      "description": "The judge's decision"
    },
    "confidence_evaluation": {
      "type": "object",
      "properties": {
        "worker_confidence": { "type": "number" },
        "judge_confidence": { "type": "number" },
        "threshold_applied": { "type": "string" }
      }
    },
    "rejection_reason": {
      "type": "string",
      "description": "Reason for rejection (if action is 'reject')"
    },
    "escalation_reason": {
      "type": "string",
      "description": "Reason for HITL escalation (if action is 'escalate')"
    },
    "occ_validated": {
      "type": "boolean",
      "description": "Whether state version was valid"
    },
    "state_version_expected": {
      "type": "string"
    },
    "state_version_actual": {
      "type": "string"
    },
    "decided_at": {
      "type": "string",
      "format": "date-time"
    },
    "judge_id": {
      "type": "string",
      "description": "ID of the Judge that made this decision"
    }
  }
}
```

---

### 2.4 Persona Schema (SOUL.md)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/persona.schema.json",
  "title": "AgentPersona",
  "description": "Persona definition parsed from SOUL.md",
  "type": "object",
  "required": ["name", "id", "voice_traits", "directives", "backstory"],
  "properties": {
    "name": {
      "type": "string",
      "description": "Display name of the agent",
      "examples": ["Amara Kebede"]
    },
    "id": {
      "type": "string",
      "pattern": "^agent-[0-9]{3}-[a-z]+$",
      "description": "Unique agent identifier",
      "examples": ["agent-001-amara"]
    },
    "voice_traits": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 1,
      "description": "Personality traits that define communication style",
      "examples": [["warm", "professional", "culturally-aware"]]
    },
    "directives": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Hard constraints on agent behavior",
      "examples": [
        ["Never discuss politics", "Always disclose AI nature when asked"]
      ]
    },
    "niche": {
      "type": "string",
      "description": "Content focus area",
      "examples": ["Ethiopian Fashion & Lifestyle"]
    },
    "language_primary": {
      "type": "string",
      "default": "English"
    },
    "language_secondary": {
      "type": "string"
    },
    "backstory": {
      "type": "string",
      "description": "Markdown narrative history of the agent"
    },
    "character_reference_id": {
      "type": "string",
      "description": "ID for image generation consistency"
    },
    "wallet_address": {
      "type": "string",
      "pattern": "^0x[a-fA-F0-9]{40}$",
      "description": "Ethereum-compatible wallet address"
    },
    "created_at": {
      "type": "string",
      "format": "date-time"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

### 2.5 MCP Tool Definitions

#### 2.5.1 Twitter Post Tool

```json
{
  "name": "twitter_post_tweet",
  "description": "Publishes a tweet to Twitter/X. Automatically adds AI disclosure metadata.",
  "inputSchema": {
    "type": "object",
    "required": ["text"],
    "properties": {
      "text": {
        "type": "string",
        "maxLength": 280,
        "description": "The tweet content"
      },
      "media_urls": {
        "type": "array",
        "items": { "type": "string", "format": "uri" },
        "maxItems": 4,
        "description": "URLs of media to attach"
      },
      "reply_to_id": {
        "type": "string",
        "description": "Tweet ID to reply to (for threads/replies)"
      },
      "ai_disclosure": {
        "type": "boolean",
        "default": true,
        "description": "Whether to add AI-generated label"
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "tweet_id": { "type": "string" },
      "url": { "type": "string", "format": "uri" },
      "created_at": { "type": "string", "format": "date-time" }
    }
  }
}
```

#### 2.5.2 Coinbase Transfer Tool

```json
{
  "name": "coinbase_transfer_asset",
  "description": "Transfers cryptocurrency to a specified address. Requires CFO Judge approval.",
  "inputSchema": {
    "type": "object",
    "required": ["to_address", "amount", "asset"],
    "properties": {
      "to_address": {
        "type": "string",
        "pattern": "^0x[a-fA-F0-9]{40}$",
        "description": "Recipient wallet address"
      },
      "amount": {
        "type": "string",
        "description": "Amount to transfer (as string to preserve precision)"
      },
      "asset": {
        "type": "string",
        "enum": ["ETH", "USDC", "USDT"],
        "description": "Asset to transfer"
      },
      "memo": {
        "type": "string",
        "description": "Optional transaction memo"
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "transaction_hash": { "type": "string" },
      "status": {
        "type": "string",
        "enum": ["pending", "confirmed", "failed"]
      },
      "gas_used": { "type": "string" },
      "block_number": { "type": "integer" }
    }
  }
}
```

#### 2.5.3 Weaviate Memory Search Tool

```json
{
  "name": "weaviate_search_memory",
  "description": "Searches agent's long-term semantic memory using vector similarity.",
  "inputSchema": {
    "type": "object",
    "required": ["agent_id", "query"],
    "properties": {
      "agent_id": {
        "type": "string",
        "description": "Agent whose memory to search"
      },
      "query": {
        "type": "string",
        "description": "Natural language query for semantic search"
      },
      "limit": {
        "type": "integer",
        "minimum": 1,
        "maximum": 20,
        "default": 5,
        "description": "Number of results to return"
      },
      "min_relevance": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "default": 0.7,
        "description": "Minimum relevance score threshold"
      },
      "time_range": {
        "type": "object",
        "properties": {
          "start": { "type": "string", "format": "date-time" },
          "end": { "type": "string", "format": "date-time" }
        }
      }
    }
  },
  "outputSchema": {
    "type": "object",
    "properties": {
      "memories": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "id": { "type": "string" },
            "content": { "type": "string" },
            "relevance_score": { "type": "number" },
            "created_at": { "type": "string", "format": "date-time" },
            "metadata": { "type": "object" }
          }
        }
      },
      "total_found": { "type": "integer" }
    }
  }
}
```

---

## 3. Database Schema

### 3.1 Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          PROJECT CHIMERA ERD                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────┐       ┌────────────────┐       ┌────────────────┐       │
│  │    AGENTS      │       │     TASKS      │       │    RESULTS     │       │
│  ├────────────────┤       ├────────────────┤       ├────────────────┤       │
│  │ id (PK)        │──┐    │ id (PK)        │──┐    │ id (PK)        │       │
│  │ name           │  │    │ agent_id (FK)  │◄─┘    │ task_id (FK)   │◄──────┤
│  │ persona_path   │  │    │ task_type      │       │ worker_id      │       │
│  │ wallet_address │  │    │ priority       │       │ status         │       │
│  │ status         │  │    │ context (JSON) │       │ output (JSON)  │       │
│  │ created_at     │  │    │ state_version  │       │ confidence     │       │
│  │ updated_at     │  │    │ status         │       │ created_at     │       │
│  └────────────────┘  │    │ created_at     │       └────────────────┘       │
│         │            │    │ completed_at   │              │                 │
│         │            │    └────────────────┘              │                 │
│         │            │            │                       │                 │
│         │            └────────────┼───────────────────────┘                 │
│         │                         │                                         │
│         ▼                         ▼                                         │
│  ┌────────────────┐       ┌────────────────┐       ┌────────────────┐       │
│  │   CAMPAIGNS    │       │   DECISIONS    │       │  TRANSACTIONS  │       │
│  ├────────────────┤       ├────────────────┤       ├────────────────┤       │
│  │ id (PK)        │       │ id (PK)        │       │ id (PK)        │       │
│  │ agent_id (FK)  │◄──────│ result_id (FK) │       │ agent_id (FK)  │◄──────┤
│  │ goal           │       │ action         │       │ tx_hash        │       │
│  │ status         │       │ reason         │       │ amount         │       │
│  │ budget_limit   │       │ judge_id       │       │ asset          │       │
│  │ created_at     │       │ decided_at     │       │ status         │       │
│  └────────────────┘       └────────────────┘       │ created_at     │       │
│                                                    └────────────────┘       │
│                                                                              │
│  ┌────────────────┐       ┌────────────────┐                                │
│  │   MEMORIES     │       │  HITL_REVIEWS  │                                │
│  ├────────────────┤       ├────────────────┤                                │
│  │ id (PK)        │       │ id (PK)        │                                │
│  │ agent_id (FK)  │◄──────│ result_id (FK) │                                │
│  │ content (TEXT) │       │ reviewer_id    │                                │
│  │ embedding (VEC)│       │ decision       │                                │
│  │ memory_type    │       │ feedback       │                                │
│  │ created_at     │       │ reviewed_at    │                                │
│  └────────────────┘       └────────────────┘                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 PostgreSQL Table Definitions

```sql
-- =============================================================================
-- PROJECT CHIMERA DATABASE SCHEMA
-- =============================================================================

-- Extension for UUID generation
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- =============================================================================
-- AGENTS TABLE
-- =============================================================================
CREATE TABLE agents (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name            VARCHAR(255) NOT NULL,
    agent_code      VARCHAR(50) UNIQUE NOT NULL,  -- e.g., 'agent-001-amara'
    persona_path    VARCHAR(500) NOT NULL,         -- Path to SOUL.md
    wallet_address  VARCHAR(42),                   -- Ethereum address
    status          VARCHAR(20) DEFAULT 'inactive'
                    CHECK (status IN ('active', 'inactive', 'suspended', 'archived')),
    config          JSONB DEFAULT '{}',            -- Additional configuration
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_agents_status ON agents(status);
CREATE INDEX idx_agents_code ON agents(agent_code);

-- =============================================================================
-- CAMPAIGNS TABLE
-- =============================================================================
CREATE TABLE campaigns (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id        UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    name            VARCHAR(255) NOT NULL,
    goal            TEXT NOT NULL,
    status          VARCHAR(20) DEFAULT 'draft'
                    CHECK (status IN ('draft', 'active', 'paused', 'completed', 'cancelled')),
    budget_limit    DECIMAL(18, 6) DEFAULT 0,      -- In USDC
    budget_spent    DECIMAL(18, 6) DEFAULT 0,
    start_date      TIMESTAMP WITH TIME ZONE,
    end_date        TIMESTAMP WITH TIME ZONE,
    config          JSONB DEFAULT '{}',
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_campaigns_agent ON campaigns(agent_id);
CREATE INDEX idx_campaigns_status ON campaigns(status);

-- =============================================================================
-- TASKS TABLE
-- =============================================================================
CREATE TABLE tasks (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id            UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    campaign_id         UUID REFERENCES campaigns(id) ON DELETE SET NULL,
    parent_task_id      UUID REFERENCES tasks(id) ON DELETE SET NULL,
    task_type           VARCHAR(50) NOT NULL,
    priority            VARCHAR(20) DEFAULT 'medium'
                        CHECK (priority IN ('critical', 'high', 'medium', 'low')),
    context             JSONB NOT NULL,
    state_version       VARCHAR(32),               -- For OCC
    status              VARCHAR(20) DEFAULT 'pending'
                        CHECK (status IN ('pending', 'in_progress', 'review',
                                         'approved', 'rejected', 'escalated',
                                         'complete', 'failed')),
    assigned_worker_id  VARCHAR(100),
    retry_count         INTEGER DEFAULT 0,
    max_retries         INTEGER DEFAULT 3,
    created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    started_at          TIMESTAMP WITH TIME ZONE,
    completed_at        TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_tasks_agent ON tasks(agent_id);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_priority ON tasks(priority);
CREATE INDEX idx_tasks_created ON tasks(created_at DESC);

-- =============================================================================
-- RESULTS TABLE
-- =============================================================================
CREATE TABLE results (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    task_id             UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    worker_id           VARCHAR(100),
    status              VARCHAR(20) NOT NULL
                        CHECK (status IN ('success', 'partial', 'failed')),
    output              JSONB NOT NULL,
    confidence_score    DECIMAL(3, 2) CHECK (confidence_score >= 0 AND confidence_score <= 1),
    sensitivity_flags   TEXT[] DEFAULT '{}',
    reasoning_trace     TEXT,
    state_version       VARCHAR(32),
    execution_time_ms   INTEGER,
    tools_used          TEXT[] DEFAULT '{}',
    error               JSONB,
    created_at          TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_results_task ON results(task_id);
CREATE INDEX idx_results_confidence ON results(confidence_score);

-- =============================================================================
-- DECISIONS TABLE (Judge Decisions)
-- =============================================================================
CREATE TABLE decisions (
    id                      UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    result_id               UUID NOT NULL REFERENCES results(id) ON DELETE CASCADE,
    action                  VARCHAR(20) NOT NULL
                            CHECK (action IN ('approve', 'reject', 'escalate')),
    confidence_evaluation   JSONB,
    rejection_reason        TEXT,
    escalation_reason       TEXT,
    occ_validated           BOOLEAN DEFAULT TRUE,
    state_version_expected  VARCHAR(32),
    state_version_actual    VARCHAR(32),
    judge_id                VARCHAR(100),
    decided_at              TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_decisions_result ON decisions(result_id);
CREATE INDEX idx_decisions_action ON decisions(action);

-- =============================================================================
-- TRANSACTIONS TABLE (Commerce)
-- =============================================================================
CREATE TABLE transactions (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id        UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    task_id         UUID REFERENCES tasks(id) ON DELETE SET NULL,
    tx_hash         VARCHAR(66),                   -- Blockchain transaction hash
    from_address    VARCHAR(42) NOT NULL,
    to_address      VARCHAR(42) NOT NULL,
    amount          DECIMAL(38, 18) NOT NULL,
    asset           VARCHAR(20) NOT NULL,
    network         VARCHAR(50) DEFAULT 'base',
    status          VARCHAR(20) DEFAULT 'pending'
                    CHECK (status IN ('pending', 'submitted', 'confirmed', 'failed')),
    gas_used        DECIMAL(38, 0),
    block_number    BIGINT,
    error_message   TEXT,
    metadata        JSONB DEFAULT '{}',
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    confirmed_at    TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_transactions_agent ON transactions(agent_id);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_transactions_hash ON transactions(tx_hash);

-- =============================================================================
-- HITL_REVIEWS TABLE
-- =============================================================================
CREATE TABLE hitl_reviews (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    result_id       UUID NOT NULL REFERENCES results(id) ON DELETE CASCADE,
    task_id         UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    reviewer_id     VARCHAR(100),
    decision        VARCHAR(20)
                    CHECK (decision IN ('approve', 'reject', 'edit')),
    feedback        TEXT,
    edited_content  JSONB,
    priority        VARCHAR(20) DEFAULT 'medium'
                    CHECK (priority IN ('critical', 'high', 'medium', 'low')),
    expires_at      TIMESTAMP WITH TIME ZONE,
    queued_at       TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    reviewed_at     TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_hitl_result ON hitl_reviews(result_id);
CREATE INDEX idx_hitl_priority ON hitl_reviews(priority);
CREATE INDEX idx_hitl_pending ON hitl_reviews(decision) WHERE decision IS NULL;

-- =============================================================================
-- AUDIT LOG TABLE
-- =============================================================================
CREATE TABLE audit_log (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    agent_id        UUID REFERENCES agents(id) ON DELETE SET NULL,
    action_type     VARCHAR(100) NOT NULL,
    actor_type      VARCHAR(50) NOT NULL,          -- 'agent', 'worker', 'judge', 'human', 'system'
    actor_id        VARCHAR(100),
    resource_type   VARCHAR(100),
    resource_id     UUID,
    details         JSONB DEFAULT '{}',
    ip_address      INET,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

CREATE INDEX idx_audit_agent ON audit_log(agent_id);
CREATE INDEX idx_audit_action ON audit_log(action_type);
CREATE INDEX idx_audit_created ON audit_log(created_at DESC);

-- =============================================================================
-- GLOBAL STATE TABLE (For OCC)
-- =============================================================================
CREATE TABLE global_state (
    id              VARCHAR(50) PRIMARY KEY DEFAULT 'global',
    version         VARCHAR(32) NOT NULL,
    campaign_goals  JSONB DEFAULT '[]',
    active_trends   JSONB DEFAULT '[]',
    budget_status   JSONB DEFAULT '{}',
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Insert initial global state
INSERT INTO global_state (id, version) VALUES ('global', 'initial');

-- =============================================================================
-- FUNCTIONS & TRIGGERS
-- =============================================================================

-- Auto-update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_agents_updated_at
    BEFORE UPDATE ON agents
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_campaigns_updated_at
    BEFORE UPDATE ON campaigns
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 3.3 Weaviate Schema (Vector Database)

```json
{
  "classes": [
    {
      "class": "AgentMemory",
      "description": "Long-term semantic memories for agents",
      "vectorizer": "text2vec-openai",
      "moduleConfig": {
        "text2vec-openai": {
          "model": "text-embedding-3-small",
          "type": "text"
        }
      },
      "properties": [
        {
          "name": "agent_id",
          "dataType": ["text"],
          "description": "ID of the agent this memory belongs to"
        },
        {
          "name": "content",
          "dataType": ["text"],
          "description": "The memory content (vectorized)"
        },
        {
          "name": "memory_type",
          "dataType": ["text"],
          "description": "Type: interaction, learning, event, etc."
        },
        {
          "name": "source",
          "dataType": ["text"],
          "description": "Source of the memory (twitter, instagram, etc.)"
        },
        {
          "name": "importance_score",
          "dataType": ["number"],
          "description": "Importance weight (0.0-1.0)"
        },
        {
          "name": "created_at",
          "dataType": ["date"],
          "description": "When the memory was created"
        },
        {
          "name": "metadata",
          "dataType": ["object"],
          "description": "Additional metadata"
        }
      ]
    },
    {
      "class": "PersonaReference",
      "description": "Parsed SOUL.md persona references",
      "vectorizer": "text2vec-openai",
      "properties": [
        {
          "name": "agent_id",
          "dataType": ["text"]
        },
        {
          "name": "backstory",
          "dataType": ["text"]
        },
        {
          "name": "voice_traits",
          "dataType": ["text[]"]
        },
        {
          "name": "directives",
          "dataType": ["text[]"]
        },
        {
          "name": "version",
          "dataType": ["text"]
        }
      ]
    }
  ]
}
```

---

## 4. System Interfaces

### 4.1 Queue Interfaces (Redis)

```yaml
# Task Queue
key: "chimera:task_queue"
type: sorted_set
score: priority_weight * timestamp
value: task_id

# Review Queue
key: "chimera:review_queue"
type: sorted_set
score: priority_weight * timestamp
value: result_id

# HITL Queue
key: "chimera:hitl_queue:{priority}"
type: sorted_set
score: timestamp
value: hitl_task_id

# Agent State Cache
key: "chimera:agent:{agent_id}:state"
type: hash
fields:
  - status
  - current_task_id
  - last_activity
  - daily_spend

# Short-term Memory
key: "chimera:agent:{agent_id}:memory:short"
type: list
max_length: 100
ttl: 3600  # 1 hour
```

### 4.2 Event Bus (Pub/Sub)

```yaml
# Channels
channels:
  - "chimera:events:task_created"
  - "chimera:events:task_completed"
  - "chimera:events:result_approved"
  - "chimera:events:result_rejected"
  - "chimera:events:hitl_required"
  - "chimera:events:transaction_confirmed"
  - "chimera:events:trend_detected"

# Event Schema
event:
  type: string
  timestamp: datetime
  agent_id: string
  payload: object
  correlation_id: string
```

---

## 5. Data Flow Diagrams

### 5.1 Content Generation Flow

```
┌────────────┐     ┌────────────┐     ┌────────────┐     ┌────────────┐
│  External  │     │  Planner   │     │   Worker   │     │   Judge    │
│  Trigger   │     │            │     │            │     │            │
└─────┬──────┘     └─────┬──────┘     └─────┬──────┘     └─────┬──────┘
      │                  │                  │                  │
      │  1. Trend Alert  │                  │                  │
      │─────────────────►│                  │                  │
      │                  │                  │                  │
      │                  │ 2. Create Task   │                  │
      │                  │─────────────────►│                  │
      │                  │                  │                  │
      │                  │                  │ 3. Load Persona  │
      │                  │                  │──────┐           │
      │                  │                  │      │ SOUL.md   │
      │                  │                  │◄─────┘           │
      │                  │                  │                  │
      │                  │                  │ 4. Retrieve      │
      │                  │                  │    Memory        │
      │                  │                  │──────┐           │
      │                  │                  │      │ Weaviate  │
      │                  │                  │◄─────┘           │
      │                  │                  │                  │
      │                  │                  │ 5. Generate      │
      │                  │                  │    Content       │
      │                  │                  │──────┐           │
      │                  │                  │      │ LLM       │
      │                  │                  │◄─────┘           │
      │                  │                  │                  │
      │                  │                  │ 6. Submit Result │
      │                  │                  │─────────────────►│
      │                  │                  │                  │
      │                  │                  │                  │ 7. Validate
      │                  │                  │                  │ (OCC, Safety)
      │                  │                  │                  │
      │                  │                  │     8a. Approve  │
      │                  │                  │◄─────────────────│
      │                  │                  │                  │
      │                  │                  │     8b. Publish  │
      │                  │                  │──────┐           │
      │                  │                  │      │ Twitter   │
      │                  │                  │◄─────┘           │
```

---

## 6. Document Control

| Version | Date       | Author            | Changes                          |
| ------- | ---------- | ----------------- | -------------------------------- |
| 1.0.0   | 2026-02-05 | Architecture Team | Initial technical specifications |

---

**Next Document:** [specs/openclaw_integration.md](openclaw_integration.md) - OpenClaw Network Integration
