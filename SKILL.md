---
name: voice-memo-sync
description: |
  Sync, transcribe, and intelligently organize Apple Voice Memos and audio/video files.
  Supports: Voice Memos app, iCloud directories, uploaded files, YouTube/Bilibili URLs.
  同步、转录、智能整理Apple语音备忘录及音视频文件，支持多种输入源。
version: 1.1.0
author: Ying Wen
homepage: https://github.com/ying-wen/voice-memo-sync
metadata:
  openclaw:
    emoji: "🎙️"
    os: ["darwin"]
    requires:
      bins: ["ffprobe", "python3"]
      optional_bins: ["whisper", "remindctl", "summarize"]
    install:
      - id: ffmpeg
        kind: brew
        formula: ffmpeg
        bins: ["ffprobe", "ffmpeg"]
        label: "Install FFmpeg (required)"
      - id: whisper
        kind: brew  
        formula: openai-whisper
        bins: ["whisper"]
        label: "Install Whisper (optional fallback)"
      - id: remindctl
        kind: brew
        formula: steipete/tap/remindctl
        bins: ["remindctl"]
        label: "Install remindctl (optional, for Reminders)"
      - id: summarize
        kind: brew
        formula: steipete/tap/summarize
        bins: ["summarize"]
        label: "Install summarize (optional, for URLs)"
---

# Voice Memo Sync

Intelligently sync and organize voice memos and audio/video with AI-powered analysis.

## When to Use

✅ **USE this skill when:**
- User says "同步语音备忘录" / "sync voice memos"
- User says "整理录音" / "process recording"  
- User says "开完会了" / "会议结束" / "meeting done"
- User shares a voice/video file (.m4a, .mp3, .wav, .qta, .mp4, .mov)
- User shares a transcript text (NoteGPT export, etc.)
- User shares a YouTube/Bilibili URL
- User wants to sync from iCloud directory

## Input Sources

### 1. Apple Voice Memos (Default)
```
路径: ~/Library/Group Containers/group.com.apple.VoiceMemos.shared/Recordings/
格式: .qta (新录音), .m4a (旧录音)
特点: 自动提取Apple原生转录
```

### 2. iCloud Directory (User Configurable)
```yaml
# 在 config/voice-memo-sync.yaml 中配置
sources:
  icloud:
    enabled: true
    paths:
      - "~/Library/Mobile Documents/com~apple~CloudDocs/Recordings"
      - "~/Library/Mobile Documents/com~apple~CloudDocs/会议录音"
      - "~/Library/Mobile Documents/com~apple~CloudDocs/Downloads"
    watch_patterns: ["*.m4a", "*.mp3", "*.mp4", "*.wav", "*.mov"]
```

### 3. Direct File Upload
```
用户: "帮我整理这个录音" + [附件]
支持: .m4a, .mp3, .wav, .mp4, .mov, .qta
```

### 4. Transcript Text
```
用户: "整理这段会议记录: [粘贴文本]"
用户: [发送NoteGPT导出的.txt文件]
特点: 跳过转录，直接LLM整理
```

### 5. Video URLs
```
用户: "处理这个视频 https://youtube.com/..."
用户: "整理这个B站视频 https://bilibili.com/video/..."
工具: summarize CLI (需安装)
```

## Quick Commands

### Sync Latest Voice Memo
```
用户: "同步下最新的录音"
```

### Sync from iCloud
```
用户: "同步iCloud里的录音"
用户: "检查下会议录音文件夹有没有新文件"
```

### Process Specific File
```
用户: "处理 ~/Downloads/meeting.mp4"
```

### Process URL
```
用户: "整理这个视频 https://www.youtube.com/watch?v=..."
```

## Transcription Priority

1. **Apple Native** (Voice Memos only): 从.qta/.m4a的meta atom提取
2. **NoteGPT/Text**: 用户提供的转录文本
3. **summarize CLI**: YouTube/Bilibili字幕提取
4. **Whisper Local** (fallback): 本地运行，隐私安全
5. **External API** (optional): 火山引擎/OpenAI Whisper API

## Output Structure

写入Apple Notes的内容结构：
```
🎙️ [智能生成的标题]
📅 时间 | ⏱️ 时长 | 🏷️ #标签1 #标签2

📌 核心摘要
[一段话总结]

🎯 关键要点
• 要点1
• 要点2

💡 深度分析与反思
[结合用户背景USER.md的个性化分析]

📋 行动建议
• TODO 1
• TODO 2

🔗 相关联系
[与用户其他项目/记忆的关联]

💬 金句摘录 (可选)
• "引用1"
• "引用2"

---
📝 原始转录
[灰色小字，放最后]
```

## Configuration

创建 `~/.openclaw/workspace/config/voice-memo-sync.yaml`：

```yaml
# 输入源配置
sources:
  # Apple Voice Memos (always enabled)
  voice_memos:
    enabled: true
  
  # iCloud目录监控
  icloud:
    enabled: true
    paths:
      - "~/Library/Mobile Documents/com~apple~CloudDocs/Recordings"
      - "~/Library/Mobile Documents/com~apple~CloudDocs/会议录音"
    watch_patterns: ["*.m4a", "*.mp3", "*.mp4", "*.wav"]
    
  # 自定义本地目录
  local:
    enabled: false
    paths:
      - "~/Downloads"
    watch_patterns: ["*.m4a", "*.mp3"]

# 转录配置
transcription:
  priority: ["apple", "text", "summarize", "whisper-local", "whisper-api"]
  whisper_model: "small"
  language: "zh"

# Apple Notes配置
notes:
  folder: "语音备忘录"
  include_quotes: true      # 是否包含金句摘录
  include_original: true    # 是否包含原始转录

# Reminders配置  
reminders:
  enabled: true
  list: "Reminders"
  auto_create: true

# 用户上下文 (自动读取)
context:
  user_profile: "USER.md"
  memory: "MEMORY.md"
  soul: "SOUL.md"
```

## Processing Flow

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT SOURCES                        │
├─────────────────────────────────────────────────────────┤
│  Voice Memos │ iCloud Dir │ Upload │ Text │ URL        │
└───────┬──────┴─────┬──────┴───┬────┴──┬───┴──┬─────────┘
        │            │          │       │      │
        ▼            ▼          ▼       ▼      ▼
┌─────────────────────────────────────────────────────────┐
│                  TRANSCRIPTION LAYER                    │
├─────────────────────────────────────────────────────────┤
│ Apple Native │ Direct Text │ summarize │ Whisper       │
└───────┬──────┴──────┬──────┴─────┬─────┴───┬───────────┘
        │             │            │         │
        └─────────────┴────────────┴─────────┘
                        │
                        ▼
        ┌─────────────────────────────┐
        │   LLM INTELLIGENT ANALYSIS  │
        │  ─────────────────────────  │
        │  • Read USER.md (背景)      │
        │  • Read MEMORY.md (记忆)    │
        │  • Reconstruct garbled text │
        │  • Extract key points       │
        │  • Generate insights        │
        │  • Identify TODOs           │
        │  • Find connections         │
        └──────────────┬──────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
┌─────────────────┐       ┌─────────────────┐
│   Apple Notes   │       │    Reminders    │
│  (Structured)   │       │    (TODOs)      │
│  + #tags        │       │                 │
└─────────────────┘       └─────────────────┘
```

## Privacy & Security

⚠️ **隐私保护设计:**
- 所有转录默认在本地完成
- 外部API需用户明确配置
- 不存储任何API密钥在代码中
- 用户记忆文件只在本地读取
- iCloud路径仅访问用户指定目录

## Examples

### Example 1: Sync iCloud Recordings
```
用户: "检查下iCloud会议录音文件夹"

Agent动作:
1. 读取config获取iCloud路径
2. 扫描 ~/Library/Mobile Documents/com~apple~CloudDocs/会议录音/
3. 找到新文件 meeting-2026-03-08.m4a
4. 用Whisper转录
5. LLM整理 + 写入Notes + 创建Reminders
```

### Example 2: Process YouTube Video
```
用户: "整理下这个视频 https://youtube.com/watch?v=xxx"

Agent动作:
1. 调用 summarize "URL" --youtube auto --extract-only
2. 获取字幕/转录
3. LLM深度整理（结合USER.md背景）
4. 写入Apple Notes（带#标签）
5. 提取TODO写入Reminders
```

### Example 3: Process NoteGPT Export
```
用户: [发送 NoteGPT_xxx.txt 文件]
用户: "整理下这个转录"

Agent动作:
1. 读取txt文件内容
2. 跳过转录步骤
3. LLM深度整理
4. 写入Notes + Reminders
```

## Scripts

| 脚本 | 用途 |
|------|------|
| `scripts/extract-apple-transcript.py` | 提取Apple原生转录 |
| `scripts/sync-icloud-recordings.sh` | 同步iCloud目录 |
| `scripts/create-apple-note.sh` | 创建Apple Notes |

## Changelog

### v1.1.0 (2026-03-08)
- 新增iCloud目录同步支持
- 新增YouTube/Bilibili URL处理
- 新增NoteGPT转录文本处理
- 优化LLM整理输出结构
- 增加金句摘录功能
