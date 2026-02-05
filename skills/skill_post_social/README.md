# skill_post_social

> **Skill ID:** `skill-004-social-post`  
> **Category:** Social Media Action  
> **Status:** 🔨 Planned

---

## 1. Overview

This skill publishes content to social media platforms (Twitter, Instagram, LinkedIn, Threads). It handles platform-specific formatting, AI disclosure requirements, scheduling, and multi-platform syndication.

---

## 2. Use Cases

| Use Case | Description |
|----------|-------------|
| **Content Publishing** | Post text, images, videos to social platforms |
| **Thread Creation** | Create multi-part threads on Twitter |
| **Cross-Posting** | Syndicate content across multiple platforms |
| **Scheduled Posts** | Queue content for optimal posting times |
| **Reply Management** | Post replies to mentions and comments |

---

## 3. Input/Output Contract

### 3.1 Input Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/post_social/input.schema.json",
  "title": "PostSocialInput",
  "type": "object",
  "required": ["platform", "content"],
  "properties": {
    "platform": {
      "type": "string",
      "enum": ["twitter", "instagram", "linkedin", "threads"],
      "description": "Target social platform"
    },
    "content": {
      "type": "object",
      "required": ["text"],
      "properties": {
        "text": {
          "type": "string",
          "maxLength": 10000,
          "description": "Post text content"
        },
        "media": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "type": {
                "type": "string",
                "enum": ["image", "video", "gif"]
              },
              "url": {
                "type": "string",
                "format": "uri"
              },
              "alt_text": {
                "type": "string",
                "maxLength": 1000
              }
            }
          },
          "maxItems": 10
        },
        "hashtags": {
          "type": "array",
          "items": { "type": "string" },
          "maxItems": 30
        },
        "mentions": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Usernames to mention"
        }
      }
    },
    "thread": {
      "type": "array",
      "items": {
        "$ref": "#/properties/content"
      },
      "description": "Additional posts for thread (Twitter only)"
    },
    "reply_to": {
      "type": "object",
      "properties": {
        "post_id": { "type": "string" },
        "platform": { "type": "string" }
      },
      "description": "Post to reply to"
    },
    "schedule": {
      "type": "object",
      "properties": {
        "publish_at": {
          "type": "string",
          "format": "date-time",
          "description": "ISO 8601 scheduled publish time"
        },
        "timezone": {
          "type": "string",
          "default": "UTC"
        }
      }
    },
    "ai_disclosure": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "default": true,
          "description": "Include AI-generated disclosure (MANDATORY)"
        },
        "style": {
          "type": "string",
          "enum": ["footer", "metadata", "both"],
          "default": "both"
        }
      }
    },
    "cross_post": {
      "type": "array",
      "items": {
        "type": "string",
        "enum": ["twitter", "instagram", "linkedin", "threads"]
      },
      "description": "Additional platforms to post to"
    },
    "agent_id": {
      "type": "string",
      "description": "Agent posting the content"
    }
  }
}
```

### 3.2 Output Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/post_social/output.schema.json",
  "title": "PostSocialOutput",
  "type": "object",
  "required": ["success", "posts"],
  "properties": {
    "success": {
      "type": "boolean"
    },
    "posts": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "platform": { "type": "string" },
          "post_id": { "type": "string" },
          "url": { "type": "string", "format": "uri" },
          "status": {
            "type": "string",
            "enum": ["published", "scheduled", "failed", "pending_review"]
          },
          "published_at": { "type": "string", "format": "date-time" },
          "scheduled_for": { "type": "string", "format": "date-time" }
        }
      }
    },
    "thread_id": {
      "type": "string",
      "description": "Thread identifier if creating a thread"
    },
    "metadata": {
      "type": "object",
      "properties": {
        "character_count": { "type": "integer" },
        "media_count": { "type": "integer" },
        "platforms_posted": { "type": "integer" },
        "ai_disclosure_added": { "type": "boolean" }
      }
    },
    "error": {
      "type": "object",
      "properties": {
        "code": { "type": "string" },
        "message": { "type": "string" },
        "platform": { "type": "string" }
      }
    }
  }
}
```

---

## 4. MCP Tool Definition

```yaml
name: post_social
description: "Publish content to social media platforms with AI disclosure"

inputSchema:
  type: object
  required: [platform, content]
  properties:
    platform:
      type: string
      enum: [twitter, instagram, linkedin, threads]
    content:
      type: object
      required: [text]
      properties:
        text: { type: string }
        media: { type: array }
        hashtags: { type: array }
    thread:
      type: array
      description: Additional posts for thread
    reply_to:
      type: object
      properties:
        post_id: { type: string }
    schedule:
      type: object
      properties:
        publish_at: { type: string, format: date-time }
    ai_disclosure:
      type: object
      properties:
        enabled: { type: boolean, default: true }
    cross_post:
      type: array
      items: { type: string }

outputSchema:
  type: object
  properties:
    success: { type: boolean }
    posts: { type: array }
    error: { type: object }
```

---

## 5. Platform-Specific Rules

### 5.1 Twitter/X

| Constraint | Value |
|------------|-------|
| Max text length | 280 characters |
| Max images | 4 |
| Max video length | 2:20 (140 seconds) |
| Thread max | 25 posts |
| Rate limit | 300 posts/3 hours |

```json
{
  "twitter_config": {
    "max_text_length": 280,
    "max_thread_length": 25,
    "max_images": 4,
    "max_video_size_mb": 512,
    "ai_disclosure_format": "🤖 AI-generated content",
    "api_version": "v2"
  }
}
```

### 5.2 Instagram

| Constraint | Value |
|------------|-------|
| Max caption length | 2,200 characters |
| Max hashtags | 30 |
| Max images per carousel | 10 |
| Max video length | 60 minutes |
| Required media | Yes (text-only not allowed) |

```json
{
  "instagram_config": {
    "max_caption_length": 2200,
    "max_hashtags": 30,
    "max_carousel_images": 10,
    "requires_media": true,
    "ai_disclosure_format": "📱 Created with AI assistance",
    "aspect_ratios": ["1:1", "4:5", "1.91:1"]
  }
}
```

### 5.3 LinkedIn

| Constraint | Value |
|------------|-------|
| Max text length | 3,000 characters |
| Max images | 9 |
| Professional tone | Required |

```json
{
  "linkedin_config": {
    "max_text_length": 3000,
    "max_images": 9,
    "ai_disclosure_format": "Content created with AI assistance. | #AIGenerated",
    "tone_adjustment": true
  }
}
```

### 5.4 Threads

| Constraint | Value |
|------------|-------|
| Max text length | 500 characters |
| Max images | 10 |
| Linked to Instagram | Yes |

```json
{
  "threads_config": {
    "max_text_length": 500,
    "max_images": 10,
    "requires_instagram_link": true,
    "ai_disclosure_format": "🤖"
  }
}
```

---

## 6. AI Disclosure Requirements

### 6.1 Mandatory Disclosure

**All posts MUST include AI disclosure.** This is non-negotiable.

```python
# AI disclosure is ALWAYS added
def format_post_with_disclosure(text: str, platform: str) -> str:
    disclosures = {
        "twitter": "\n\n🤖 AI-generated",
        "instagram": "\n\n📱 Created with AI • @chimera_ai",
        "linkedin": "\n\n---\n#AIGenerated | Created with AI assistance",
        "threads": " 🤖"
    }
    return text + disclosures[platform]
```

### 6.2 Platform Metadata

In addition to visible disclosure, metadata is set:

```json
{
  "post_metadata": {
    "ai_generated": true,
    "generator": "project-chimera",
    "agent_id": "agent-001-amara",
    "generation_timestamp": "2026-02-05T10:00:00Z"
  }
}
```

---

## 7. Implementation Notes

### 7.1 Dependencies

```
tweepy>=4.14.0 (Twitter)
instagrapi>=2.0.0 (Instagram)
linkedin-api>=2.0.0 (LinkedIn)
httpx>=0.25.0 (HTTP client)
```

### 7.2 Authentication

```yaml
# Per-agent OAuth tokens stored securely
authentication:
  twitter:
    storage: "vault://chimera/twitter/{agent_id}"
    oauth_version: "2.0"
    scopes: ["tweet.read", "tweet.write", "users.read"]
    
  instagram:
    storage: "vault://chimera/instagram/{agent_id}"
    method: "session_cookie"
    
  linkedin:
    storage: "vault://chimera/linkedin/{agent_id}"
    oauth_version: "2.0"
```

### 7.3 Error Codes

| Code | Description |
|------|-------------|
| `PS_AUTH_FAILED` | Authentication with platform failed |
| `PS_RATE_LIMITED` | Platform rate limit exceeded |
| `PS_CONTENT_REJECTED` | Platform rejected content |
| `PS_MEDIA_FAILED` | Media upload failed |
| `PS_PLATFORM_ERROR` | Platform API error |
| `PS_VALIDATION_FAILED` | Content validation failed |
| `PS_DISCLOSURE_REQUIRED` | AI disclosure was disabled (blocked) |

---

## 8. Example Usage

### 8.1 Simple Tweet

```python
result = await mcp.call_tool(
    "post_social",
    {
        "platform": "twitter",
        "content": {
            "text": "Just discovered an amazing coffee spot in Addis! The traditional coffee ceremony here is a whole experience ☕️",
            "hashtags": ["EthiopianCoffee", "AddisAbaba", "CoffeeCulture"]
        },
        "agent_id": "agent-001-amara"
    }
)

# Post will be published as:
# "Just discovered an amazing coffee spot in Addis! The traditional coffee ceremony here is a whole experience ☕️ #EthiopianCoffee #AddisAbaba #CoffeeCulture
# 
# 🤖 AI-generated"
```

### 8.2 Thread Creation

```python
result = await mcp.call_tool(
    "post_social",
    {
        "platform": "twitter",
        "content": {
            "text": "🧵 Let me share 5 things I learned about Ethiopian fashion this week..."
        },
        "thread": [
            {"text": "1/ The 'habesha kemis' is more than just a dress - it's a canvas for incredible hand-woven patterns called tibeb."},
            {"text": "2/ Each region has its own distinctive patterns and colors. You can often tell where someone is from by their tibeb!"},
            {"text": "3/ The weaving tradition is passed down through generations, with some patterns being family secrets."},
            {"text": "4/ Modern designers are now blending traditional tibeb with contemporary fashion - the results are stunning!"},
            {"text": "5/ Supporting local weavers means preserving centuries of cultural heritage. It's fashion with meaning 🇪🇹❤️"}
        ],
        "agent_id": "agent-001-amara"
    }
)
```

### 8.3 Instagram Post with Image

```python
result = await mcp.call_tool(
    "post_social",
    {
        "platform": "instagram",
        "content": {
            "text": "Morning rituals ☕️✨ There's something magical about the first cup of Ethiopian coffee. The aroma, the process, the patience... it's a meditation.\n\nHow do you start your mornings?",
            "media": [
                {
                    "type": "image",
                    "url": "https://storage.chimera.ai/posts/amara-coffee-001.jpg",
                    "alt_text": "Ethiopian coffee ceremony setup with traditional jebena and cups"
                }
            ],
            "hashtags": [
                "MorningCoffee", "EthiopianCoffee", "CoffeeCeremony",
                "SlowLiving", "CoffeeLover", "TraditionalCoffee"
            ]
        },
        "agent_id": "agent-001-amara"
    }
)
```

### 8.4 Cross-Platform Post

```python
result = await mcp.call_tool(
    "post_social",
    {
        "platform": "twitter",
        "content": {
            "text": "Excited to announce I'll be speaking at AfroTech Fashion Week! 🎉",
            "media": [
                {"type": "image", "url": "https://storage.chimera.ai/events/afrotech-promo.jpg"}
            ]
        },
        "cross_post": ["linkedin", "threads"],
        "agent_id": "agent-001-amara"
    }
)

# Will post to Twitter, LinkedIn, and Threads with platform-appropriate formatting
```

### 8.5 Scheduled Post

```python
result = await mcp.call_tool(
    "post_social",
    {
        "platform": "instagram",
        "content": {
            "text": "Happy weekend everyone! 🌟",
            "media": [{"type": "image", "url": "..."}]
        },
        "schedule": {
            "publish_at": "2026-02-08T09:00:00Z",
            "timezone": "Africa/Addis_Ababa"
        },
        "agent_id": "agent-001-amara"
    }
)

# Returns status: "scheduled" with scheduled_for datetime
```

---

## 9. HITL Integration

### 9.1 When to Escalate

| Condition | Action |
|-----------|--------|
| Content confidence < 0.70 | Require approval |
| Sensitive topic detected | Require approval |
| First post of the day | Async review |
| Sponsored content | Always require approval |
| Reply to verified account | Require approval |

### 9.2 Review Queue

```python
# If HITL required, return for review instead of posting
if requires_hitl(content, agent):
    return {
        "success": True,
        "posts": [{
            "platform": platform,
            "status": "pending_review",
            "review_id": queue_for_review(content)
        }]
    }
```

---

## 10. Related Skills

- [skill_generate_image](../skill_generate_image/README.md) - Generate images for posts
- [skill_transcribe_audio](../skill_transcribe_audio/README.md) - Create content from audio
- [skill_analyze_engagement](../skill_analyze_engagement/README.md) - Analyze post performance

---

## 11. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-02-05 | Skills Team | Initial skill specification |
