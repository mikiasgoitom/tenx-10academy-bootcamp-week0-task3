# Project Chimera: Tooling Strategy

> **Version:** 1.0.0  
> **Status:** Draft  
> **Last Updated:** February 5, 2026

---

## 1. Overview

This document outlines the MCP (Model Context Protocol) server selection and tooling strategy for Project Chimera. It covers:

- Development-time MCP servers (filesystem, git, etc.)
- Runtime MCP servers (social media, commerce, etc.)
- Custom MCP server development guidelines
- Integration patterns and security considerations

---

## 2. MCP Architecture Primer

### 2.1 What is MCP?

The **Model Context Protocol (MCP)** is a universal interface for AI agents to interact with external systems. It provides three primitives:

| Primitive | Direction | Purpose |
|-----------|-----------|---------|
| **Tools** | Agent → Server | Execute actions (post tweet, send payment) |
| **Resources** | Agent ← Server | Read data (fetch mentions, get balance) |
| **Prompts** | Server → Agent | Inject context (persona instructions) |

### 2.2 MCP in Project Chimera

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CHIMERA MCP TOPOLOGY                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        CHIMERA AGENT RUNTIME                           │  │
│  │                                                                         │  │
│  │    ┌─────────┐    ┌─────────┐    ┌─────────┐                           │  │
│  │    │ Planner │    │ Worker  │    │  Judge  │                           │  │
│  │    └────┬────┘    └────┬────┘    └────┬────┘                           │  │
│  │         │              │              │                                 │  │
│  │         └──────────────┼──────────────┘                                 │  │
│  │                        │                                                │  │
│  │                        ▼                                                │  │
│  │               ┌────────────────┐                                        │  │
│  │               │   MCP CLIENT   │                                        │  │
│  │               │                │                                        │  │
│  │               └───────┬────────┘                                        │  │
│  └───────────────────────┼────────────────────────────────────────────────┘  │
│                          │                                                   │
│          ┌───────────────┼───────────────┬───────────────┐                  │
│          │               │               │               │                  │
│          ▼               ▼               ▼               ▼                  │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │  Twitter MCP │ │ Coinbase MCP │ │ YouTube MCP  │ │ Memory MCP   │       │
│  │    Server    │ │    Server    │ │    Server    │ │    Server    │       │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Development-Time MCP Servers

### 3.1 Essential Dev Servers

| Server | Repository | Purpose | Priority |
|--------|------------|---------|----------|
| **filesystem-mcp** | `modelcontextprotocol/servers/filesystem` | Read/write project files | ⭐ Critical |
| **git-mcp** | `modelcontextprotocol/servers/git` | Commit, branch, diff operations | ⭐ Critical |
| **fetch-mcp** | `modelcontextprotocol/servers/fetch` | HTTP requests for research | High |
| **sqlite-mcp** | `modelcontextprotocol/servers/sqlite` | Local database testing | Medium |
| **postgres-mcp** | `modelcontextprotocol/servers/postgres` | Database operations | High |

### 3.2 Configuration

```json
// .mcp/servers.json (development)
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./"],
      "env": {}
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git"],
      "env": {}
    },
    "fetch": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-fetch"],
      "env": {}
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "${POSTGRES_URL}"
      }
    }
  }
}
```

### 3.3 Tool Reference

#### filesystem-mcp

```yaml
tools:
  - read_file:
      description: "Read contents of a file"
      input: { path: string }
      output: { content: string }
      
  - write_file:
      description: "Write content to a file"
      input: { path: string, content: string }
      output: { success: boolean }
      
  - list_directory:
      description: "List files in a directory"
      input: { path: string }
      output: { entries: string[] }
```

#### git-mcp

```yaml
tools:
  - git_status:
      description: "Get repository status"
      output: { staged: string[], unstaged: string[], untracked: string[] }
      
  - git_commit:
      description: "Create a commit"
      input: { message: string }
      output: { commit_hash: string }
      
  - git_diff:
      description: "Show changes between commits"
      input: { from: string?, to: string? }
      output: { diff: string }
```

---

## 4. Runtime MCP Servers

### 4.1 Social Media Servers

| Server | Source | Platforms | Status |
|--------|--------|-----------|--------|
| **twitter-mcp** | Custom build | Twitter/X | 🔨 TODO |
| **instagram-mcp** | Custom build | Instagram | 🔨 TODO |
| **linkedin-mcp** | Custom build | LinkedIn | 🔨 TODO |

#### twitter-mcp (Custom)

```yaml
# chimera-twitter-mcp specification
name: chimera-twitter-mcp
version: 1.0.0

tools:
  - twitter_post_tweet:
      description: "Post a tweet with optional media"
      input:
        text: string (max 280)
        media_urls: string[]?
        reply_to_id: string?
        ai_disclosure: boolean (default: true)
      output:
        tweet_id: string
        url: string
        created_at: datetime
      
  - twitter_reply:
      description: "Reply to a tweet"
      input:
        tweet_id: string
        text: string
      output:
        reply_id: string
        
  - twitter_delete:
      description: "Delete a tweet"
      input:
        tweet_id: string
      output:
        success: boolean

resources:
  - twitter://mentions/{agent_id}:
      description: "Recent mentions of the agent"
      returns: list[Tweet]
      
  - twitter://timeline/{agent_id}:
      description: "Agent's timeline"
      returns: list[Tweet]
      
  - twitter://followers/{agent_id}:
      description: "Follower count and list"
      returns: FollowerData
```

### 4.2 Commerce Servers

| Server | Source | Purpose | Status |
|--------|--------|---------|--------|
| **coinbase-mcp** | Official Coinbase | AgentKit integration | ✅ Available |
| **stripe-mcp** | Community | Fiat payments | 🔍 Evaluate |

#### coinbase-mcp (Official)

```yaml
# Coinbase AgentKit MCP
name: @coinbase/agentkit-mcp
source: https://github.com/coinbase/agentkit

tools:
  - wallet_create:
      description: "Create a new MPC wallet"
      output:
        wallet_id: string
        address: string
        
  - wallet_get_balance:
      description: "Get wallet balance"
      input:
        wallet_id: string
        asset: enum[ETH, USDC, USDT]
      output:
        balance: string
        
  - transfer_asset:
      description: "Transfer cryptocurrency"
      input:
        from_wallet_id: string
        to_address: string
        amount: string
        asset: enum[ETH, USDC, USDT]
      output:
        transaction_hash: string
        status: enum[pending, confirmed, failed]
        
  - trade_asset:
      description: "Trade between assets"
      input:
        wallet_id: string
        from_asset: string
        to_asset: string
        amount: string
      output:
        trade_id: string
        executed_price: string

resources:
  - coinbase://wallet/{wallet_id}/transactions:
      description: "Transaction history"
      returns: list[Transaction]
```

**Security Configuration:**

```yaml
# coinbase-mcp security settings
security:
  # Maximum single transaction (USDC)
  max_transaction_amount: "100.00"
  
  # Daily spend limit per agent
  daily_spend_limit: "500.00"
  
  # Transactions above this require CFO Judge
  cfo_review_threshold: "50.00"
  
  # Allowed destination addresses (whitelist)
  allowed_destinations:
    - pattern: "^0x[a-fA-F0-9]{40}$"
      requires_verification: true
```

### 4.3 Content & Media Servers

| Server | Source | Purpose | Status |
|--------|--------|---------|--------|
| **youtube-mcp** | Custom build | Video download/analysis | 🔨 TODO |
| **ffmpeg-mcp** | Community | Media processing | 🔍 Evaluate |
| **replicate-mcp** | Community | Image generation | 🔍 Evaluate |

### 4.4 Memory & RAG Servers

| Server | Source | Purpose | Status |
|--------|--------|---------|--------|
| **weaviate-mcp** | Custom build | Vector memory | 🔨 TODO |
| **redis-mcp** | Community | Short-term cache | ✅ Available |

#### weaviate-mcp (Custom)

```yaml
name: chimera-weaviate-mcp
version: 1.0.0

tools:
  - memory_search:
      description: "Semantic search in agent memory"
      input:
        agent_id: string
        query: string
        limit: integer (default: 5)
        min_relevance: float (default: 0.7)
      output:
        memories: list[Memory]
        
  - memory_store:
      description: "Store a new memory"
      input:
        agent_id: string
        content: string
        memory_type: enum[interaction, learning, event]
        importance: float
      output:
        memory_id: string
        
  - memory_delete:
      description: "Delete a memory"
      input:
        memory_id: string
      output:
        success: boolean

resources:
  - weaviate://agent/{agent_id}/memories:
      description: "All memories for an agent"
      returns: list[Memory]
```

---

## 5. Custom MCP Server Development

### 5.1 When to Build Custom

Build a custom MCP server when:
- No suitable community server exists
- You need Chimera-specific business logic
- Security requirements demand custom controls
- Performance optimization is critical

### 5.2 Development Template

```python
# skills/skill_template/mcp_server.py
"""
Template for Chimera MCP Server Development
"""

from mcp_server import MCPServer, Tool, Resource

class ChimeraSkillServer(MCPServer):
    """Base class for Chimera MCP servers."""
    
    def __init__(self, config: dict):
        super().__init__()
        self.config = config
        self._register_tools()
        self._register_resources()
    
    def _register_tools(self):
        """Register tools with the MCP server."""
        raise NotImplementedError
    
    def _register_resources(self):
        """Register resources with the MCP server."""
        raise NotImplementedError
    
    async def validate_request(self, request: dict) -> bool:
        """
        Validate incoming request.
        Override for custom validation logic.
        """
        return True
```

### 5.3 Testing Custom Servers

```python
# tests/mcp/test_skill_server.py
import pytest
from mcp_client import MCPClient

@pytest.fixture
def mcp_client():
    client = MCPClient()
    client.connect("skill-server", "localhost:5000")
    return client

async def test_tool_invocation(mcp_client):
    result = await mcp_client.call_tool(
        name="skill_name",
        arguments={"input": "test"}
    )
    assert result.success is True
```

---

## 6. Server Selection Matrix

### 6.1 Decision Framework

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MCP SERVER SELECTION DECISION TREE                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Need external integration?                                                 │
│         │                                                                    │
│         ├── YES ──► Official MCP server exists?                              │
│         │               │                                                    │
│         │               ├── YES ──► Meets security requirements?             │
│         │               │               │                                    │
│         │               │               ├── YES ──► USE OFFICIAL             │
│         │               │               │                                    │
│         │               │               └── NO ──► BUILD WRAPPER             │
│         │               │                                                    │
│         │               └── NO ──► Community server exists?                  │
│         │                               │                                    │
│         │                               ├── YES ──► Actively maintained?     │
│         │                               │               │                    │
│         │                               │               ├── YES ──► EVALUATE │
│         │                               │               │                    │
│         │                               │               └── NO ──► BUILD     │
│         │                               │                                    │
│         │                               └── NO ──► BUILD CUSTOM              │
│         │                                                                    │
│         └── NO ──► Use internal implementation                               │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Priority Matrix

| Server | Priority | Source | Timeline |
|--------|----------|--------|----------|
| coinbase-mcp | P0 | Official | Week 1 |
| filesystem-mcp | P0 | Official | Week 1 |
| git-mcp | P0 | Official | Week 1 |
| twitter-mcp | P0 | Custom | Week 2-3 |
| weaviate-mcp | P0 | Custom | Week 2-3 |
| youtube-mcp | P1 | Custom | Week 4 |
| instagram-mcp | P1 | Custom | Week 4 |
| replicate-mcp | P2 | Evaluate | Week 5 |

---

## 7. Security Guidelines

### 7.1 Credential Management

```yaml
# NEVER do this in code
api_key = "sk-1234567890"  # ❌ HARDCODED

# ALWAYS use environment variables
api_key = os.environ["TWITTER_API_KEY"]  # ✅ SECURE
```

### 7.2 MCP Server Security Checklist

- [ ] **Input Validation**: Validate all tool inputs against schemas
- [ ] **Rate Limiting**: Implement per-agent rate limits
- [ ] **Audit Logging**: Log all tool invocations
- [ ] **Error Sanitization**: Don't leak sensitive info in errors
- [ ] **Timeout Enforcement**: Kill long-running operations
- [ ] **Principle of Least Privilege**: Request minimal permissions

### 7.3 Network Security

```yaml
# MCP server network configuration
network:
  # Only allow connections from Chimera runtime
  allowed_origins:
    - "chimera-runtime.internal"
    
  # TLS required
  tls:
    enabled: true
    cert_path: "/etc/ssl/mcp/cert.pem"
    key_path: "/etc/ssl/mcp/key.pem"
    
  # Connection limits
  max_connections: 100
  connection_timeout_seconds: 30
```

---

## 8. Integration Patterns

### 8.1 Tool Chaining

```python
async def create_and_post_content(agent: Agent, topic: str):
    """
    Chain multiple MCP tools to complete a workflow.
    """
    # 1. Search memory for context
    memories = await mcp.call_tool(
        "memory_search",
        {"agent_id": agent.id, "query": topic}
    )
    
    # 2. Generate content (internal LLM call, not MCP)
    content = await generate_content(topic, memories)
    
    # 3. Post to Twitter
    result = await mcp.call_tool(
        "twitter_post_tweet",
        {"text": content.text, "ai_disclosure": True}
    )
    
    # 4. Store memory of the interaction
    await mcp.call_tool(
        "memory_store",
        {
            "agent_id": agent.id,
            "content": f"Posted about {topic}",
            "memory_type": "event"
        }
    )
    
    return result
```

### 8.2 Error Handling

```python
from mcp_client import MCPError, MCPTimeoutError, MCPRateLimitError

async def safe_tool_call(tool_name: str, args: dict, retries: int = 3):
    """Robust MCP tool invocation with retry logic."""
    for attempt in range(retries):
        try:
            return await mcp.call_tool(tool_name, args)
            
        except MCPTimeoutError:
            logger.warning(f"Timeout on {tool_name}, attempt {attempt + 1}")
            await asyncio.sleep(2 ** attempt)
            
        except MCPRateLimitError:
            logger.warning(f"Rate limited on {tool_name}")
            await asyncio.sleep(60)
            
        except MCPError as e:
            logger.error(f"MCP error: {e}")
            raise
    
    raise MaxRetriesExceeded(f"Failed after {retries} attempts")
```

---

## 9. Monitoring & Observability

### 9.1 Metrics to Track

| Metric | Type | Alert Threshold |
|--------|------|-----------------|
| Tool invocation latency | Histogram | p99 > 5s |
| Tool error rate | Counter | > 5% over 5min |
| Rate limit hits | Counter | > 10/min |
| Connection pool usage | Gauge | > 80% |

### 9.2 Logging

```python
# Structured logging for MCP operations
logger.info(
    "mcp_tool_invoked",
    extra={
        "tool_name": tool_name,
        "agent_id": agent.id,
        "latency_ms": latency,
        "success": success,
        "error_code": error_code
    }
)
```

---

## 10. Roadmap

### Phase 1: Foundation (Weeks 1-2)
- [ ] Set up development MCP servers (filesystem, git, postgres)
- [ ] Integrate Coinbase AgentKit MCP
- [ ] Create MCP client wrapper for Chimera

### Phase 2: Core Integrations (Weeks 3-4)
- [ ] Build twitter-mcp custom server
- [ ] Build weaviate-mcp custom server
- [ ] Implement tool chaining patterns

### Phase 3: Expansion (Weeks 5-6)
- [ ] Build youtube-mcp for content ingestion
- [ ] Evaluate and integrate image generation MCP
- [ ] Build instagram-mcp

### Phase 4: Optimization (Weeks 7-8)
- [ ] Performance tuning
- [ ] Advanced error handling
- [ ] Monitoring and alerting

---

## 11. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-02-05 | Tooling Team | Initial tooling strategy |

---

**Related Documents:**
- [specs/technical.md](../specs/technical.md) - API schemas for MCP tools
- [CLAUDE.md](../CLAUDE.md) - AI assistant context
