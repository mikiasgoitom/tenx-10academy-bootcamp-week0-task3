# skill_transcribe_audio

> **Skill ID:** `skill-002-audio-transcribe`  
> **Category:** Content Processing  
> **Status:** 🔨 Planned

---

## 1. Overview

This skill transcribes audio files into text using Whisper (or compatible models). It supports multiple languages, speaker diarization, and timestamp generation. It is a critical component in the content analysis pipeline.

---

## 2. Use Cases

| Use Case | Description |
|----------|-------------|
| **Video Analysis** | Transcribe YouTube videos for content analysis |
| **Trend Research** | Extract topics from popular podcasts/videos |
| **Caption Generation** | Generate captions for agent-created content |
| **Voice Notes** | Process voice memo inputs from operators |

---

## 3. Input/Output Contract

### 3.1 Input Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/transcribe_audio/input.schema.json",
  "title": "TranscribeAudioInput",
  "type": "object",
  "required": ["audio_source"],
  "properties": {
    "audio_source": {
      "oneOf": [
        {
          "type": "object",
          "required": ["file_path"],
          "properties": {
            "file_path": {
              "type": "string",
              "description": "Path to local audio file"
            }
          }
        },
        {
          "type": "object",
          "required": ["url"],
          "properties": {
            "url": {
              "type": "string",
              "format": "uri",
              "description": "URL to audio file"
            }
          }
        }
      ],
      "description": "Audio source (file path or URL)"
    },
    "language": {
      "type": "string",
      "description": "ISO 639-1 language code (auto-detect if omitted)",
      "examples": ["en", "am", "sw", "fr"]
    },
    "model": {
      "type": "string",
      "enum": ["whisper-tiny", "whisper-base", "whisper-small", "whisper-medium", "whisper-large"],
      "default": "whisper-base",
      "description": "Whisper model size"
    },
    "enable_diarization": {
      "type": "boolean",
      "default": false,
      "description": "Enable speaker diarization (who said what)"
    },
    "include_timestamps": {
      "type": "boolean",
      "default": true,
      "description": "Include word/segment timestamps"
    },
    "output_format": {
      "type": "string",
      "enum": ["text", "srt", "vtt", "json"],
      "default": "json",
      "description": "Output format"
    },
    "max_duration_seconds": {
      "type": "integer",
      "minimum": 1,
      "maximum": 7200,
      "default": 1800,
      "description": "Maximum audio duration (30 min default)"
    }
  }
}
```

### 3.2 Output Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "chimera://skills/transcribe_audio/output.schema.json",
  "title": "TranscribeAudioOutput",
  "type": "object",
  "required": ["success", "transcription"],
  "properties": {
    "success": {
      "type": "boolean",
      "description": "Whether transcription completed successfully"
    },
    "transcription": {
      "type": "object",
      "properties": {
        "text": {
          "type": "string",
          "description": "Full transcription text"
        },
        "segments": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "id": { "type": "integer" },
              "start": { "type": "number", "description": "Start time in seconds" },
              "end": { "type": "number", "description": "End time in seconds" },
              "text": { "type": "string" },
              "speaker": { "type": "string", "description": "Speaker ID if diarization enabled" },
              "confidence": { "type": "number", "minimum": 0, "maximum": 1 }
            }
          }
        },
        "language": {
          "type": "string",
          "description": "Detected or specified language"
        },
        "duration_seconds": {
          "type": "number",
          "description": "Total audio duration"
        }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "model_used": { "type": "string" },
        "processing_time_seconds": { "type": "number" },
        "word_count": { "type": "integer" },
        "speaker_count": { "type": "integer" }
      }
    },
    "srt_content": {
      "type": "string",
      "description": "SRT subtitle content (if format=srt)"
    },
    "vtt_content": {
      "type": "string",
      "description": "WebVTT subtitle content (if format=vtt)"
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
name: transcribe_audio
description: "Transcribe audio to text using Whisper with optional diarization"

inputSchema:
  type: object
  required: [audio_source]
  properties:
    audio_source:
      oneOf:
        - type: object
          properties:
            file_path: { type: string }
        - type: object
          properties:
            url: { type: string, format: uri }
    language:
      type: string
      description: ISO 639-1 code
    model:
      type: string
      enum: [whisper-tiny, whisper-base, whisper-small, whisper-medium, whisper-large]
      default: whisper-base
    enable_diarization:
      type: boolean
      default: false
    include_timestamps:
      type: boolean
      default: true
    output_format:
      type: string
      enum: [text, srt, vtt, json]
      default: json

outputSchema:
  type: object
  properties:
    success: { type: boolean }
    transcription:
      type: object
      properties:
        text: { type: string }
        segments: { type: array }
        language: { type: string }
    metadata: { type: object }
    error: { type: object }
```

---

## 5. Implementation Notes

### 5.1 Dependencies

```
openai-whisper>=20231117
torch>=2.0
pyannote.audio (for diarization)
ffmpeg (system dependency)
```

### 5.2 Model Selection Guide

| Model | VRAM | Speed | Accuracy | Use Case |
|-------|------|-------|----------|----------|
| tiny | ~1GB | Fastest | Basic | Quick previews |
| base | ~1GB | Fast | Good | Most tasks |
| small | ~2GB | Medium | Better | Important content |
| medium | ~5GB | Slow | High | High-quality needs |
| large | ~10GB | Slowest | Highest | Critical accuracy |

### 5.3 Language Support

Whisper supports 100+ languages. Key languages for Chimera:

| Language | Code | Native Support |
|----------|------|----------------|
| English | en | ✅ Excellent |
| Amharic | am | ⚠️ Limited |
| Swahili | sw | ⚠️ Limited |
| French | fr | ✅ Excellent |
| Arabic | ar | ✅ Good |

### 5.4 Error Codes

| Code | Description |
|------|-------------|
| `TR_INVALID_SOURCE` | Audio source not found or invalid |
| `TR_UNSUPPORTED_FORMAT` | Audio format not supported |
| `TR_TOO_LONG` | Audio exceeds maximum duration |
| `TR_PROCESSING_FAILED` | Transcription failed |
| `TR_DIARIZATION_FAILED` | Speaker diarization failed |
| `TR_OUT_OF_MEMORY` | Model too large for available memory |

---

## 6. Example Usage

```python
# Transcribe audio from downloaded YouTube video
result = await mcp.call_tool(
    "transcribe_audio",
    {
        "audio_source": {"file_path": "/tmp/video_audio.mp3"},
        "language": "en",
        "model": "whisper-base",
        "include_timestamps": True,
        "output_format": "json"
    }
)

if result.success:
    print(f"Transcription: {result.transcription.text[:500]}...")
    print(f"Duration: {result.transcription.duration_seconds}s")
    print(f"Word count: {result.metadata.word_count}")
    
    # Access segments with timestamps
    for segment in result.transcription.segments[:5]:
        print(f"[{segment.start:.2f}s] {segment.text}")
else:
    print(f"Error: {result.error.message}")
```

### 6.1 Pipeline Example

```python
# Full pipeline: Download → Transcribe → Analyze
async def analyze_youtube_video(url: str):
    # Step 1: Download
    download = await mcp.call_tool(
        "youtube_download",
        {"url": url, "extract_audio": True, "output_format": "mp3"}
    )
    
    if not download.success:
        return {"error": "Download failed"}
    
    # Step 2: Transcribe
    transcription = await mcp.call_tool(
        "transcribe_audio",
        {
            "audio_source": {"file_path": download.file_path},
            "model": "whisper-small",
            "enable_diarization": True
        }
    )
    
    if not transcription.success:
        return {"error": "Transcription failed"}
    
    # Step 3: Return combined result
    return {
        "video_metadata": download.metadata,
        "transcription": transcription.transcription.text,
        "speakers": transcription.metadata.speaker_count
    }
```

---

## 7. Related Skills

- [skill_download_youtube](../skill_download_youtube/README.md) - Download source audio
- [skill_analyze_sentiment](../skill_analyze_sentiment/README.md) - Analyze transcription sentiment
- [skill_extract_topics](../skill_extract_topics/README.md) - Extract topics from text

---

## 8. Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-02-05 | Skills Team | Initial skill specification |
