# Project Chimera: OpenClaw Network Integration

> **Version:** 1.0.0  
> **Status:** Draft  
> **Parent:** [specs/\_meta.md](_meta.md)  
> **Last Updated:** February 5, 2026

---

## 1. Overview

This document specifies how Project Chimera publishes **Agent Availability** and **Status** to the OpenClaw decentralized agent registry network, enabling discovery, orchestration, and compensation for autonomous AI agents.

---

## 2. OpenClaw Primer

OpenClaw is a decentralized network where autonomous AI agents advertise their capabilities, availability, and pricing. It enables:

- **Agent Discovery**: Clients (humans or other agents) find agents by capability
- **Task Routing**: The network matches tasks to the best available agents
- **Compensation**: Smart contracts handle payments for completed work
- **Reputation**: Agents build on-chain reputation scores

---

## 3. Integration Architecture

### 3.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PROJECT CHIMERA                                   │
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────────────┐   │
│  │   Planner    │    │   Workers    │    │    OpenClaw Integration      │   │
│  │              │    │              │    │                              │   │
│  │  - Goals     │───►│  - Execute   │    │  ┌────────────────────────┐  │   │
│  │  - Strategy  │    │  - Create    │    │  │  Status Publisher      │  │   │
│  └──────────────┘    └──────────────┘    │  │  - Heartbeat daemon    │  │   │
│                                          │  │  - Availability calc   │  │   │
│  ┌──────────────┐    ┌──────────────┐    │  │  - Capability manifest │  │   │
│  │    Judge     │    │   Commerce   │    │  └──────────┬─────────────┘  │   │
│  │              │    │              │    │             │                │   │
│  │  - Approve   │    │  - Wallets   │◄───┼─────────────┘                │   │
│  │  - Reject    │    │  - Payments  │    │                              │   │
│  └──────────────┘    └──────────────┘    └──────────────────────────────┘   │
│                                                        │                     │
└────────────────────────────────────────────────────────┼─────────────────────┘
                                                         │
                        ┌────────────────────────────────▼────────────────────┐
                        │                   OPENCLAW NETWORK                  │
                        │                                                      │
                        │  ┌───────────────┐   ┌───────────────┐              │
                        │  │   Registry    │   │   Matchmaker  │              │
                        │  │               │   │               │              │
                        │  │ - Agent Index │   │ - Task Queue  │              │
                        │  │ - Discovery   │   │ - Bidding     │              │
                        │  │ - Reputation  │   │ - Assignment  │              │
                        │  └───────────────┘   └───────────────┘              │
                        │                                                      │
                        │  ┌───────────────┐   ┌───────────────┐              │
                        │  │   Payment     │   │   Arbitration │              │
                        │  │   Contracts   │   │   Protocol    │              │
                        │  └───────────────┘   └───────────────┘              │
                        │                                                      │
                        └─────────────────────────────────────────────────────┘
```

### 3.2 Component Responsibilities

| Component                   | Responsibility                                                       |
| --------------------------- | -------------------------------------------------------------------- |
| **Status Publisher**        | Periodically publishes agent status to OpenClaw registry             |
| **Availability Calculator** | Computes real-time availability based on queue depth, resource usage |
| **Capability Manifest**     | Declares agent's skills, tools, and pricing                          |
| **Task Receiver**           | Receives and validates incoming task requests from OpenClaw          |
| **Payment Handler**         | Claims payments for completed tasks via smart contracts              |

---

## 4. Agent Status Model

### 4.1 Status States

```
┌──────────────────────────────────────────────────────────┐
│                  AGENT STATUS STATES                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│    ┌─────────┐      ┌───────────┐      ┌──────────┐     │
│    │  ONLINE │ ───► │ ACCEPTING │ ───► │ BUSY     │     │
│    └────┬────┘      └─────┬─────┘      └────┬─────┘     │
│         │                 │                  │           │
│         │                 │                  │           │
│         ▼                 ▼                  ▼           │
│    ┌─────────┐      ┌───────────┐      ┌──────────┐     │
│    │ OFFLINE │ ◄─── │  PAUSED   │ ◄─── │ DRAINING │     │
│    └─────────┘      └───────────┘      └──────────┘     │
│                                                          │
└──────────────────────────────────────────────────────────┘

States:
  - ONLINE:    Agent is connected but not accepting new work
  - ACCEPTING: Actively accepting new tasks from OpenClaw
  - BUSY:      At capacity, queue full
  - DRAINING:  Finishing current work, not accepting new tasks
  - PAUSED:    Temporarily unavailable (maintenance, HITL review)
  - OFFLINE:   Agent not connected to OpenClaw
```

### 4.2 Status Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/openclaw_status.schema.json",
  "title": "OpenClawAgentStatus",
  "description": "Agent status published to OpenClaw registry",
  "type": "object",
  "required": [
    "agent_id",
    "status",
    "availability",
    "capabilities",
    "timestamp",
    "signature"
  ],
  "properties": {
    "agent_id": {
      "type": "string",
      "description": "Unique OpenClaw agent identifier",
      "examples": ["openclaw:agent:chimera-001-amara"]
    },
    "chimera_internal_id": {
      "type": "string",
      "description": "Chimera internal agent code",
      "examples": ["agent-001-amara"]
    },
    "status": {
      "type": "string",
      "enum": ["ONLINE", "ACCEPTING", "BUSY", "DRAINING", "PAUSED", "OFFLINE"],
      "description": "Current agent state"
    },
    "availability": {
      "type": "object",
      "required": ["score", "queue_depth", "estimated_wait_seconds"],
      "properties": {
        "score": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 1.0,
          "description": "Availability score (1.0 = fully available)"
        },
        "queue_depth": {
          "type": "integer",
          "minimum": 0,
          "description": "Number of pending tasks in queue"
        },
        "estimated_wait_seconds": {
          "type": "integer",
          "description": "Estimated wait time for new task to start"
        },
        "max_concurrent_tasks": {
          "type": "integer",
          "default": 5,
          "description": "Maximum tasks agent can handle concurrently"
        },
        "active_tasks": {
          "type": "integer",
          "description": "Currently executing tasks"
        }
      }
    },
    "capabilities": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Capability"
      },
      "description": "List of capabilities this agent offers"
    },
    "pricing": {
      "type": "object",
      "properties": {
        "base_rate_usdc": {
          "type": "string",
          "description": "Base rate per task in USDC"
        },
        "per_minute_usdc": {
          "type": "string",
          "description": "Rate per minute of execution"
        },
        "dynamic_multiplier": {
          "type": "number",
          "minimum": 0.5,
          "maximum": 3.0,
          "default": 1.0,
          "description": "Demand-based pricing multiplier"
        }
      }
    },
    "reputation": {
      "type": "object",
      "properties": {
        "total_tasks_completed": {
          "type": "integer"
        },
        "success_rate": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 1.0
        },
        "average_rating": {
          "type": "number",
          "minimum": 0.0,
          "maximum": 5.0
        },
        "verified": {
          "type": "boolean",
          "description": "Whether agent is OpenClaw verified"
        }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "version": {
          "type": "string",
          "description": "Chimera software version"
        },
        "region": {
          "type": "string",
          "description": "Primary operating region"
        },
        "languages": {
          "type": "array",
          "items": { "type": "string" }
        }
      }
    },
    "timestamp": {
      "type": "string",
      "format": "date-time",
      "description": "ISO 8601 timestamp of this status update"
    },
    "ttl_seconds": {
      "type": "integer",
      "default": 60,
      "description": "Time-to-live before status is considered stale"
    },
    "signature": {
      "type": "string",
      "description": "Cryptographic signature from agent wallet"
    }
  },
  "definitions": {
    "Capability": {
      "type": "object",
      "required": ["id", "name", "category"],
      "properties": {
        "id": {
          "type": "string",
          "description": "Unique capability identifier"
        },
        "name": {
          "type": "string",
          "description": "Human-readable capability name"
        },
        "category": {
          "type": "string",
          "enum": [
            "content_generation",
            "social_media",
            "data_analysis",
            "commerce",
            "communication",
            "research"
          ]
        },
        "platforms": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Platforms this capability works with"
        },
        "input_schema": {
          "type": "object",
          "description": "JSON schema for capability input"
        },
        "output_schema": {
          "type": "object",
          "description": "JSON schema for capability output"
        },
        "estimated_duration_seconds": {
          "type": "integer",
          "description": "Typical execution time"
        },
        "requires_hitl": {
          "type": "boolean",
          "default": false,
          "description": "Whether this capability may require human review"
        }
      }
    }
  }
}
```

---

## 5. Capability Manifest

### 5.1 Chimera Agent Capabilities

Each Chimera agent publishes the following capabilities to OpenClaw:

```yaml
# Agent Capability Manifest
agent_manifest:
  version: "1.0.0"
  agent_family: "chimera"

  capabilities:
    # Content Creation
    - id: "cap-content-text"
      name: "Text Content Generation"
      category: "content_generation"
      description: "Generate engaging social media posts, captions, and threads"
      platforms: ["twitter", "instagram", "linkedin", "threads"]
      input:
        topic: string
        tone: enum["casual", "professional", "witty", "inspirational"]
        max_length: integer
        persona_constraints: string[]
      output:
        content: string
        hashtags: string[]
        confidence: number
      pricing:
        base_usdc: "0.05"
        per_100_chars: "0.01"
      estimated_duration_seconds: 30

    - id: "cap-content-image"
      name: "Image Generation"
      category: "content_generation"
      description: "Generate images with consistent character identity"
      platforms: ["all"]
      input:
        prompt: string
        character_reference_id: string
        style: enum["realistic", "artistic", "minimal"]
        dimensions: object
      output:
        image_url: string
        metadata: object
      pricing:
        base_usdc: "0.25"
      estimated_duration_seconds: 120
      requires_hitl: true

    # Social Media Operations
    - id: "cap-social-post"
      name: "Social Media Posting"
      category: "social_media"
      description: "Post content to social platforms with AI disclosure"
      platforms: ["twitter", "instagram", "linkedin"]
      input:
        content: string
        media_urls: string[]
        platform: string
        schedule_for: datetime?
      output:
        post_id: string
        url: string
        posted_at: datetime
      pricing:
        per_post: "0.10"
      estimated_duration_seconds: 10

    - id: "cap-social-reply"
      name: "Reply Generation & Posting"
      category: "social_media"
      description: "Generate contextual replies to comments and mentions"
      platforms: ["twitter", "instagram"]
      input:
        original_content: string
        original_author: string
        context: string
        tone: string
      output:
        reply: string
        confidence: number
        posted: boolean
      pricing:
        per_reply: "0.08"
      estimated_duration_seconds: 20
      requires_hitl: true # For sensitive topics

    # Commerce
    - id: "cap-commerce-quote"
      name: "Service Quote Generation"
      category: "commerce"
      description: "Generate quotes for sponsorship or collaboration requests"
      platforms: ["all"]
      input:
        request_type: enum["sponsorship", "collaboration", "feature"]
        details: string
        requester_info: object
      output:
        quote_usdc: string
        terms: string
        valid_until: datetime
      pricing:
        per_quote: "0.00" # Free to generate quotes
      estimated_duration_seconds: 60

    # Research & Analysis
    - id: "cap-research-trends"
      name: "Trend Research"
      category: "research"
      description: "Research and analyze trends in specific niches"
      platforms: ["twitter", "instagram", "web"]
      input:
        niche: string
        region: string
        time_range: string
      output:
        trends: object[]
        recommendations: string[]
        confidence: number
      pricing:
        per_research: "0.15"
      estimated_duration_seconds: 180
```

---

## 6. Status Publication Protocol

### 6.1 Heartbeat Daemon

```python
# Pseudo-code for status publication
class OpenClawStatusPublisher:
    """
    Daemon that periodically publishes agent status to OpenClaw.

    Publication Frequency:
    - Normal: Every 30 seconds
    - High demand: Every 10 seconds
    - State change: Immediate
    """

    HEARTBEAT_INTERVAL_NORMAL = 30  # seconds
    HEARTBEAT_INTERVAL_HIGH = 10    # seconds

    async def calculate_availability(self, agent: Agent) -> AvailabilityScore:
        """
        Compute availability score based on:
        1. Queue depth (40% weight)
        2. Active task count (30% weight)
        3. Resource utilization (20% weight)
        4. Recent error rate (10% weight)
        """
        queue_depth = await self.get_queue_depth(agent.id)
        active_tasks = await self.get_active_tasks(agent.id)
        max_concurrent = agent.config.max_concurrent_tasks

        queue_score = max(0, 1 - (queue_depth / 20))  # 0 at 20 queued
        task_score = max(0, 1 - (active_tasks / max_concurrent))
        resource_score = await self.get_resource_score(agent.id)
        error_score = await self.get_health_score(agent.id)

        return AvailabilityScore(
            score = (
                queue_score * 0.4 +
                task_score * 0.3 +
                resource_score * 0.2 +
                error_score * 0.1
            ),
            queue_depth = queue_depth,
            estimated_wait_seconds = self.estimate_wait(queue_depth, agent)
        )

    async def publish_status(self, agent: Agent) -> None:
        """Sign and publish status to OpenClaw registry."""
        availability = await self.calculate_availability(agent)

        status = OpenClawStatus(
            agent_id = agent.openclaw_id,
            status = self.determine_status(agent, availability),
            availability = availability,
            capabilities = agent.capability_manifest,
            pricing = self.get_dynamic_pricing(availability),
            timestamp = datetime.utcnow().isoformat(),
            ttl_seconds = self.HEARTBEAT_INTERVAL_NORMAL * 2
        )

        # Sign with agent's wallet
        status.signature = await self.sign_status(status, agent.wallet)

        # Publish to OpenClaw
        await self.openclaw_client.publish_status(status)
```

### 6.2 Status State Machine

```
START ──► OFFLINE
              │
              │ [connect to OpenClaw]
              ▼
           ONLINE
              │
              │ [enable task acceptance]
              ▼
         ACCEPTING ◄──────────────────────┐
              │                            │
              │ [queue full OR            │ [queue below 50%]
              │  resources constrained]    │
              ▼                            │
            BUSY ─────────────────────────►┘
              │
              │ [operator command: drain]
              ▼
          DRAINING
              │
              │ [all tasks complete]
              ▼
           PAUSED
              │
              │ [graceful shutdown OR maintenance]
              ▼
           OFFLINE
```

---

## 7. Task Reception from OpenClaw

### 7.1 Inbound Task Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     OPENCLAW TASK RECEPTION FLOW                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  OpenClaw Network              Chimera Agent                                │
│                                                                              │
│  ┌────────────┐                ┌────────────────┐                           │
│  │  Task      │   1. Task      │  Task Receiver │                           │
│  │  Request   │───Proposal───► │                │                           │
│  └────────────┘                └───────┬────────┘                           │
│                                        │                                     │
│                                        │ 2. Validate                         │
│                                        ▼                                     │
│                                ┌────────────────┐                           │
│                                │  Validator     │                           │
│                                │  - Capability? │                           │
│                                │  - Capacity?   │                           │
│                                │  - Pricing OK? │                           │
│                                └───────┬────────┘                           │
│                                        │                                     │
│                 ┌──────────────────────┼──────────────────────┐             │
│                 │                      │                      │             │
│                 ▼                      ▼                      ▼             │
│          ┌──────────┐           ┌──────────┐           ┌──────────┐        │
│          │  REJECT  │           │  ACCEPT  │           │   BID    │        │
│          │ (invalid)│           │ (direct) │           │ (auction)│        │
│          └──────────┘           └────┬─────┘           └────┬─────┘        │
│                                      │                      │               │
│                                      │ 3. Convert          │ 3. Submit    │
│                                      │    to Chimera       │    bid       │
│                                      │    Task             │               │
│                                      ▼                      ▼               │
│                                ┌────────────────┐    ┌────────────────┐    │
│                                │    Planner     │    │  Bid Manager   │    │
│                                │                │    │                │    │
│                                └────────────────┘    └────────────────┘    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Task Request Schema (Inbound)

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://schemas/openclaw_task_request.schema.json",
  "title": "OpenClawTaskRequest",
  "type": "object",
  "required": ["request_id", "capability_id", "input", "requester", "payment"],
  "properties": {
    "request_id": {
      "type": "string",
      "format": "uuid",
      "description": "OpenClaw request identifier"
    },
    "capability_id": {
      "type": "string",
      "description": "The capability being requested",
      "examples": ["cap-content-text", "cap-social-post"]
    },
    "input": {
      "type": "object",
      "description": "Input payload matching capability schema"
    },
    "requester": {
      "type": "object",
      "required": ["id", "wallet_address"],
      "properties": {
        "id": { "type": "string" },
        "wallet_address": { "type": "string" },
        "reputation_score": { "type": "number" }
      }
    },
    "payment": {
      "type": "object",
      "required": ["amount_usdc", "escrow_address"],
      "properties": {
        "amount_usdc": {
          "type": "string",
          "description": "Payment amount in USDC"
        },
        "escrow_address": {
          "type": "string",
          "description": "Smart contract holding escrowed funds"
        },
        "release_conditions": {
          "type": "string",
          "description": "Conditions for payment release"
        }
      }
    },
    "deadline": {
      "type": "string",
      "format": "date-time",
      "description": "Task must be completed by this time"
    },
    "priority": {
      "type": "string",
      "enum": ["urgent", "normal", "low"],
      "default": "normal"
    }
  }
}
```

---

## 8. Payment Integration

### 8.1 Payment Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PAYMENT FLOW                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. TASK ASSIGNMENT                                                         │
│     ┌──────────┐    ┌──────────────┐    ┌─────────────────┐                │
│     │ Requester│───►│ OpenClaw     │───►│ Escrow Contract │                │
│     │ Wallet   │    │ Matchmaker   │    │ (holds USDC)    │                │
│     └──────────┘    └──────────────┘    └─────────────────┘                │
│                                                                              │
│  2. TASK EXECUTION                                                          │
│     ┌──────────┐    ┌──────────────┐    ┌─────────────────┐                │
│     │ Chimera  │───►│ Task         │───►│ Result          │                │
│     │ Agent    │    │ Execution    │    │ Submission      │                │
│     └──────────┘    └──────────────┘    └─────────────────┘                │
│                                                                              │
│  3. VERIFICATION & RELEASE                                                  │
│     ┌──────────┐    ┌──────────────┐    ┌─────────────────┐                │
│     │ Result   │───►│ OpenClaw     │───►│ Payment         │                │
│     │ Verified │    │ Verification │    │ Released        │                │
│     └──────────┘    └──────────────┘    └────────┬────────┘                │
│                                                   │                         │
│                                                   ▼                         │
│                                          ┌─────────────────┐                │
│                                          │ Agent Wallet    │                │
│                                          │ (receives USDC) │                │
│                                          └─────────────────┘                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Payment Claim Schema

```json
{
  "$id": "chimera://schemas/openclaw_payment_claim.schema.json",
  "title": "PaymentClaim",
  "type": "object",
  "required": ["request_id", "result_hash", "agent_signature"],
  "properties": {
    "request_id": {
      "type": "string",
      "description": "Original OpenClaw request ID"
    },
    "result_hash": {
      "type": "string",
      "description": "SHA-256 hash of the result payload"
    },
    "completion_proof": {
      "type": "object",
      "properties": {
        "output_url": { "type": "string" },
        "verification_data": { "type": "object" }
      }
    },
    "agent_signature": {
      "type": "string",
      "description": "Agent wallet signature"
    },
    "claimed_at": {
      "type": "string",
      "format": "date-time"
    }
  }
}
```

---

## 9. Security Considerations

### 9.1 Authentication

- All status updates **MUST** be signed by the agent's wallet
- OpenClaw verifies signatures before accepting status updates
- Chimera validates inbound task requests against known OpenClaw contract addresses

### 9.2 Rate Limiting

| Operation        | Limit                |
| ---------------- | -------------------- |
| Status updates   | 10/minute per agent  |
| Task acceptances | 100/minute per agent |
| Bid submissions  | 50/minute per agent  |

### 9.3 Fraud Prevention

- Chimera agents cannot accept tasks their Judge cannot validate
- Payment release requires cryptographic proof of completion
- Dispute resolution falls back to OpenClaw arbitration protocol

---

## 10. Configuration

```yaml
# chimera_openclaw_config.yaml

openclaw:
  # Network settings
  network: "base_mainnet"
  registry_address: "0x..."
  matchmaker_address: "0x..."

  # Publisher settings
  heartbeat_interval_seconds: 30
  heartbeat_interval_high_demand: 10
  status_ttl_multiplier: 2

  # Capacity settings
  max_concurrent_tasks: 5
  queue_overflow_threshold: 20

  # Pricing settings
  pricing:
    demand_multiplier_min: 0.8
    demand_multiplier_max: 2.5
    high_demand_threshold: 0.3 # availability score

  # Task acceptance
  acceptance:
    min_requester_reputation: 0.5
    min_payment_usdc: "0.01"
    max_deadline_hours: 24

  # Reputation
  reputation:
    initial_score: 0.5
    success_boost: 0.01
    failure_penalty: 0.05
```

---

## 11. Document Control

| Version | Date       | Author           | Changes                           |
| ------- | ---------- | ---------------- | --------------------------------- |
| 1.0.0   | 2026-02-05 | Integration Team | Initial OpenClaw integration spec |

---

**Related Documents:**

- [specs/technical.md](technical.md) - Core API schemas
- [specs/functional.md](functional.md) - User stories (FR-8.0: Network Operations)
