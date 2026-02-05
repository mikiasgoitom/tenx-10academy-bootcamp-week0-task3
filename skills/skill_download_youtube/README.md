# skill_download_youtube

> **Skill ID:** `skill-001-youtube-download`  
> **Category:** Content Ingestion  
> **Status:** 🔨 Planned

---

## 1. Overview

This skill enables Chimera agents to download video and audio content from YouTube for analysis, transcription, and trend research. It wraps the popular `yt-dlp` tool and exposes it as an MCP-compatible interface.

---

## 2. Use Cases

| Use Case | Description |
|----------|-------------|
| **Trend Analysis** | Download viral videos to analyze content patterns |
| **Transcription Pipeline** | Extract audio for subsequent transcription skill |
| **Reference Material** | Gather source material for content inspiration |
| **Competitor Analysis** | Analyze competitor influencer content |

---

## 3. Input/Output Contract

### 3.1 Input Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/youtube_download/input.schema.json",
  "title": "YouTubeDownloadInput",
  "type": "object",
  "required": ["url"],
  "properties": {
    "url": {
      "type": "string",
      "format": "uri",
      "pattern": "^https?://(www\\.)?(youtube\\.com|youtu\\.be)/",
      "description": "YouTube video URL",
      "examples": [
        "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
        "https://youtu.be/dQw4w9WgXcQ"
      ]
    },
    "output_format": {
      "type": "string",
      "enum": ["mp4", "mp3", "wav", "webm"],
      "default": "mp4",
      "description": "Desired output format"
    },
    "quality": {
      "type": "string",
      "enum": ["best", "720p", "480p", "360p", "audio_only"],
      "default": "720p",
      "description": "Video quality preference"
    },
    "max_duration_seconds": {
      "type": "integer",
      "minimum": 1,
      "maximum": 3600,
      "default": 600,
      "description": "Maximum video duration to download (10 min default)"
    },
    "extract_audio": {
      "type": "boolean",
      "default": false,
      "description": "Extract audio track only"
    },
    "output_path": {
      "type": "string",
      "description": "Custom output file path (optional)"
    }
  }
}
```

### 3.2 Output Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/youtube_download/output.schema.json",
  "title": "YouTubeDownloadOutput",
  "type": "object",
  "required": ["success", "file_path", "metadata"],
  "properties": {
    "success": {
      "type": "boolean",
      "description": "Whether download completed successfully"
    },
    "file_path": {
      "type": "string",
      "description": "Path to downloaded file"
    },
    "file_size_bytes": {
      "type": "integer",
      "description": "Size of downloaded file in bytes"
    },
    "metadata": {
      "type": "object",
      "properties": {
        "title": { "type": "string" },
        "channel": { "type": "string" },
        "channel_id": { "type": "string" },
        "duration_seconds": { "type": "integer" },
        "view_count": { "type": "integer" },
        "like_count": { "type": "integer" },
        "upload_date": { "type": "string" },
        "description": { "type": "string" },
        "tags": { "type": "array", "items": { "type": "string" } },
        "thumbnail_url": { "type": "string", "format": "uri" }
      }
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
name: youtube_download
description: "Download video/audio from YouTube with metadata extraction"

inputSchema:
  type: object
  required: [url]
  properties:
    url:
      type: string
      format: uri
      description: YouTube video URL
    output_format:
      type: string
      enum: [mp4, mp3, wav, webm]
      default: mp4
    quality:
      type: string
      enum: [best, 720p, 480p, 360p, audio_only]
      default: 720p
    max_duration_seconds:
      type: integer
      default: 600
    extract_audio:
      type: boolean
      default: false

outputSchema:
  type: object
  properties:
    success: { type: boolean }
    file_path: { type: string }
    file_size_bytes: { type: integer }
    metadata: { type: object }
    error: { type: object }
```

---

## 5. Implementation Notes

### 5.1 Dependencies

```
yt-dlp>=2024.1.1
ffmpeg (system dependency)
```

### 5.2 Security Considerations

- **Content Filtering**: Do not download age-restricted or flagged content
- **Size Limits**: Enforce maximum file size (500MB default)
- **Duration Limits**: Enforce maximum video duration
- **Rate Limiting**: Respect YouTube's rate limits
- **Copyright**: Only download for analysis, not redistribution

### 5.3 Error Codes

| Code | Description |
|------|-------------|
| `YT_INVALID_URL` | URL is not a valid YouTube URL |
| `YT_VIDEO_UNAVAILABLE` | Video doesn't exist or is private |
| `YT_AGE_RESTRICTED` | Video is age-restricted |
| `YT_TOO_LONG` | Video exceeds maximum duration |
| `YT_DOWNLOAD_FAILED` | Download failed (network/other) |
| `YT_RATE_LIMITED` | YouTube rate limit hit |

---

## 6. Example Usage

```python
# Download video for analysis
result = await mcp.call_tool(
    "youtube_download",
    {
        "url": "https://www.youtube.com/watch?v=example",
        "quality": "720p",
        "max_duration_seconds": 300
    }
)

if result.success:
    print(f"Downloaded: {result.file_path}")
    print(f"Title: {result.metadata.title}")
    print(f"Views: {result.metadata.view_count}")
else:
    print(f"Error: {result.error.message}")
```

---

## 7. Related Skills

- [skill_transcribe_audio](../skill_transcribe_audio/README.md) - Transcribe downloaded audio
- [skill_extract_keyframes](../skill_extract_keyframes/README.md) - Extract key frames from video

---

## 8. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-02-05 | Skills Team | Initial skill specification |
