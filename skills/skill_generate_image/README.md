# skill_generate_image

> **Skill ID:** `skill-003-image-generate`  
> **Category:** Content Creation  
> **Status:** 🔨 Planned

---

## 1. Overview

This skill generates images using AI models (Stable Diffusion, DALL-E, Midjourney) while maintaining character consistency for agent personas. It is essential for creating visual content that aligns with each agent's unique identity defined in `SOUL.md`.

---

## 2. Use Cases

| Use Case | Description |
|----------|-------------|
| **Social Media Posts** | Create images for Instagram, Twitter posts |
| **Character Consistency** | Generate agent avatar in various poses/settings |
| **Product Visualization** | Create mockups for sponsored content |
| **Thumbnail Generation** | Generate video thumbnails |
| **Meme Creation** | Generate meme templates |

---

## 3. Input/Output Contract

### 3.1 Input Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/generate_image/input.schema.json",
  "title": "GenerateImageInput",
  "type": "object",
  "required": ["prompt"],
  "properties": {
    "prompt": {
      "type": "string",
      "minLength": 10,
      "maxLength": 2000,
      "description": "Text description of desired image",
      "examples": [
        "A professional Ethiopian woman in traditional white dress, standing in a coffee shop, warm lighting, portrait style"
      ]
    },
    "negative_prompt": {
      "type": "string",
      "maxLength": 1000,
      "description": "Elements to avoid in generation",
      "examples": [
        "blurry, low quality, distorted face, extra limbs"
      ]
    },
    "character_reference_id": {
      "type": "string",
      "description": "Agent's character reference for consistency (from SOUL.md)",
      "examples": ["char-ref-001-amara"]
    },
    "reference_images": {
      "type": "array",
      "items": { "type": "string", "format": "uri" },
      "maxItems": 4,
      "description": "URLs to reference images for style/character consistency"
    },
    "style": {
      "type": "string",
      "enum": [
        "realistic",
        "artistic",
        "minimal",
        "cinematic",
        "instagram",
        "professional",
        "cartoon",
        "anime"
      ],
      "default": "realistic",
      "description": "Visual style preset"
    },
    "dimensions": {
      "type": "object",
      "properties": {
        "width": {
          "type": "integer",
          "minimum": 256,
          "maximum": 2048,
          "default": 1024
        },
        "height": {
          "type": "integer",
          "minimum": 256,
          "maximum": 2048,
          "default": 1024
        }
      }
    },
    "aspect_ratio": {
      "type": "string",
      "enum": ["1:1", "4:5", "16:9", "9:16", "3:4", "4:3"],
      "description": "Aspect ratio (overrides dimensions if specified)"
    },
    "model": {
      "type": "string",
      "enum": [
        "stable-diffusion-xl",
        "stable-diffusion-3",
        "dall-e-3",
        "flux-pro",
        "midjourney"
      ],
      "default": "stable-diffusion-xl",
      "description": "Image generation model"
    },
    "num_images": {
      "type": "integer",
      "minimum": 1,
      "maximum": 4,
      "default": 1,
      "description": "Number of variations to generate"
    },
    "seed": {
      "type": "integer",
      "description": "Random seed for reproducibility"
    },
    "guidance_scale": {
      "type": "number",
      "minimum": 1,
      "maximum": 30,
      "default": 7.5,
      "description": "How closely to follow the prompt"
    },
    "safety_check": {
      "type": "boolean",
      "default": true,
      "description": "Run safety filter on output"
    }
  }
}
```

### 3.2 Output Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/generate_image/output.schema.json",
  "title": "GenerateImageOutput",
  "type": "object",
  "required": ["success", "images"],
  "properties": {
    "success": {
      "type": "boolean",
      "description": "Whether generation completed successfully"
    },
    "images": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "id": { "type": "string" },
          "url": { "type": "string", "format": "uri" },
          "local_path": { "type": "string" },
          "width": { "type": "integer" },
          "height": { "type": "integer" },
          "seed": { "type": "integer" },
          "safety_rating": {
            "type": "string",
            "enum": ["safe", "sensitive", "blocked"]
          }
        }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "model_used": { "type": "string" },
        "generation_time_seconds": { "type": "number" },
        "prompt_tokens": { "type": "integer" },
        "cost_usd": { "type": "string" }
      }
    },
    "character_consistency_score": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "How well the image matches character reference"
    },
    "error": {
      "type": "object",
      "properties": {
        "code": { "type": "string" },
        "message": { "type": "string" }
      }
    }
  }
}
```

---

## 4. MCP Tool Definition

```yaml
name: generate_image
description: "Generate images with AI models and character consistency support"

inputSchema:
  type: object
  required: [prompt]
  properties:
    prompt:
      type: string
      minLength: 10
      description: Text description of desired image
    negative_prompt:
      type: string
      description: Elements to avoid
    character_reference_id:
      type: string
      description: Character reference for consistency
    style:
      type: string
      enum: [realistic, artistic, minimal, cinematic, instagram, professional]
      default: realistic
    aspect_ratio:
      type: string
      enum: ["1:1", "4:5", "16:9", "9:16"]
    model:
      type: string
      enum: [stable-diffusion-xl, dall-e-3, flux-pro]
      default: stable-diffusion-xl
    num_images:
      type: integer
      minimum: 1
      maximum: 4
      default: 1
    safety_check:
      type: boolean
      default: true

outputSchema:
  type: object
  properties:
    success: { type: boolean }
    images: { type: array }
    metadata: { type: object }
    character_consistency_score: { type: number }
    error: { type: object }
```

---

## 5. Character Consistency System

### 5.1 Character Reference

Each agent has a `character_reference_id` defined in their `SOUL.md`:

```yaml
# agents/agent-001-amara/SOUL.md
character_reference:
  id: "char-ref-001-amara"
  base_images:
    - "s3://chimera-assets/characters/amara/base-portrait-1.png"
    - "s3://chimera-assets/characters/amara/base-full-body-1.png"
  physical_traits:
    ethnicity: "Ethiopian"
    age_range: "25-30"
    hair: "Long, braided, black"
    skin_tone: "Medium brown"
    distinctive_features: "High cheekbones, warm smile"
  clothing_style:
    - "Professional Ethiopian fashion"
    - "Mix of traditional and modern"
    - "White 'habesha kemis' for cultural events"
```

### 5.2 Consistency Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    CHARACTER CONSISTENCY PIPELINE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. LOAD REFERENCE                                                          │
│     ┌────────────────┐     ┌────────────────┐                               │
│     │ character_ref  │────►│ Base Images    │                               │
│     │ from SOUL.md   │     │ + Traits       │                               │
│     └────────────────┘     └───────┬────────┘                               │
│                                    │                                         │
│  2. EMBED REFERENCE                │                                         │
│     ┌────────────────┐     ┌───────▼────────┐                               │
│     │ IP-Adapter or  │◄────│ Face Embedding │                               │
│     │ LoRA weights   │     │ CLIP Embedding │                               │
│     └───────┬────────┘     └────────────────┘                               │
│             │                                                                │
│  3. GENERATE                                                                │
│     ┌───────▼────────┐     ┌────────────────┐     ┌────────────────┐       │
│     │ Conditioned    │────►│ Raw Generated  │────►│ Consistency    │       │
│     │ Generation     │     │ Image          │     │ Scoring        │       │
│     └────────────────┘     └────────────────┘     └───────┬────────┘       │
│                                                           │                 │
│  4. VALIDATE                                              │                 │
│     ┌────────────────┐     ┌────────────────┐     ┌───────▼────────┐       │
│     │ If score < 0.8 │◄────│ Safety Filter  │◄────│ Score >= 0.8   │       │
│     │ → Regenerate   │     │ (NSFW check)   │     │ → Accept       │       │
│     └────────────────┘     └────────────────┘     └────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 6. Implementation Notes

### 6.1 Dependencies

```
diffusers>=0.25.0
transformers>=4.36.0
torch>=2.0
Pillow>=10.0
accelerate>=0.25.0
ip-adapter (for character consistency)
```

### 6.2 Model Comparison

| Model | Cost | Speed | Quality | Consistency | Best For |
|-------|------|-------|---------|-------------|----------|
| SD-XL | Free* | Fast | Good | Medium | General use |
| SD-3 | Free* | Medium | Better | Good | Quality focus |
| DALL-E 3 | $0.04/img | Medium | Excellent | N/A | Text-heavy |
| Flux Pro | $0.02/img | Fast | Excellent | Good | Production |

*Self-hosted compute costs apply

### 6.3 HITL Requirements

This skill **MUST** escalate to HITL when:

| Condition | Action |
|-----------|--------|
| Character consistency score < 0.7 | Escalate for review |
| Safety filter flags content | Escalate for review |
| New character reference | Require initial approval |
| Sponsored content | Always require approval |

### 6.4 Error Codes

| Code | Description |
|------|-------------|
| `IG_INVALID_PROMPT` | Prompt too short/long or invalid |
| `IG_SAFETY_BLOCKED` | Content blocked by safety filter |
| `IG_CHARACTER_MISMATCH` | Generated image doesn't match character |
| `IG_MODEL_UNAVAILABLE` | Requested model not available |
| `IG_GENERATION_FAILED` | Generation failed (VRAM/other) |
| `IG_RATE_LIMITED` | API rate limit exceeded |

---

## 7. Example Usage

### 7.1 Basic Generation

```python
# Generate a simple image
result = await mcp.call_tool(
    "generate_image",
    {
        "prompt": "A cozy coffee shop interior with warm lighting and plants",
        "style": "instagram",
        "aspect_ratio": "4:5",
        "model": "stable-diffusion-xl"
    }
)

if result.success:
    print(f"Generated: {result.images[0].url}")
```

### 7.2 Character-Consistent Generation

```python
# Generate image with character consistency
result = await mcp.call_tool(
    "generate_image",
    {
        "prompt": "A confident woman in a modern coffee shop, holding a latte, natural lighting, professional photography",
        "character_reference_id": "char-ref-001-amara",
        "style": "instagram",
        "aspect_ratio": "1:1",
        "model": "flux-pro",
        "guidance_scale": 8.0
    }
)

if result.success:
    print(f"Consistency score: {result.character_consistency_score}")
    if result.character_consistency_score >= 0.8:
        print("Image approved for posting")
    else:
        print("Image needs regeneration or HITL review")
```

### 7.3 Instagram Post Pipeline

```python
async def create_instagram_post(agent: Agent, topic: str):
    """Full pipeline for Instagram content creation."""
    
    # 1. Generate caption
    caption = await generate_caption(agent, topic)
    
    # 2. Generate image with character
    image = await mcp.call_tool(
        "generate_image",
        {
            "prompt": caption.image_prompt,
            "character_reference_id": agent.character_ref_id,
            "style": "instagram",
            "aspect_ratio": "4:5",
            "num_images": 3  # Generate variations
        }
    )
    
    # 3. Select best image
    best_image = max(
        image.images,
        key=lambda i: i.character_consistency_score if hasattr(i, 'character_consistency_score') else 0
    )
    
    # 4. Return for Judge review
    return {
        "caption": caption.text,
        "image_url": best_image.url,
        "hashtags": caption.hashtags,
        "confidence": image.character_consistency_score
    }
```

---

## 8. Style Presets

### 8.1 Instagram Style

```json
{
  "style": "instagram",
  "negative_prompt": "blurry, low quality, watermark, text, logo, amateur, bad lighting",
  "guidance_scale": 7.5,
  "dimensions": {"width": 1080, "height": 1350}
}
```

### 8.2 Professional Style

```json
{
  "style": "professional",
  "negative_prompt": "casual, messy, unprofessional, cartoon, distorted",
  "guidance_scale": 8.0,
  "dimensions": {"width": 1024, "height": 1024}
}
```

### 8.3 Cinematic Style

```json
{
  "style": "cinematic",
  "negative_prompt": "flat lighting, amateur, snapshot, low contrast",
  "guidance_scale": 9.0,
  "dimensions": {"width": 1920, "height": 1080}
}
```

---

## 9. Related Skills

- [skill_download_youtube](../skill_download_youtube/README.md) - Download reference material
- [skill_edit_image](../skill_edit_image/README.md) - Edit generated images
- [skill_analyze_image](../skill_analyze_image/README.md) - Analyze generated content

---

## 10. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-02-05 | Skills Team | Initial skill specification |
