# Voice Memo Sync 🎙️

[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-blue)](https://github.com/openclaw/openclaw)
[![macOS](https://img.shields.io/badge/macOS-Only-lightgrey)](https://www.apple.com/macos/)
[![Apple Silicon](https://img.shields.io/badge/Apple%20Silicon-Metal%20GPU-orange)](https://support.apple.com/en-us/116943)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Intelligently sync, transcribe, and organize voice memos, audio/video files, and URLs with AI-powered analysis.**

> ⚠️ **Apple Ecosystem Only**: This skill is designed exclusively for macOS users with Apple Voice Memos, Apple Notes, and Apple Reminders integration. Linux/Windows are not supported.

Transform your voice recordings, YouTube videos, and meeting transcripts into structured, actionable notes — automatically synced to Apple Notes & Reminders.

[中文文档](README_CN.md)

## ✨ Features

- 🎙️ **Apple Native Transcription** — Extract built-in transcripts from Voice Memos (zero latency)
- 🔄 **Whisper Fallback** — Local AI transcription for recordings without native text
- 🎬 **YouTube/Bilibili Support** — Download and transcribe video content
- 📄 **Multi-format Input** — Support .m4a, .mp3, .mp4, .txt, .md, .doc, .docx, .json, .csv
- 🧠 **Smart Summarization** — LLM-powered analysis with personalized insights
- 📝 **Apple Notes Sync** — Auto-create structured notes with #tags
- ⏰ **Reminders Integration** — Extract TODOs and create reminders automatically
- 🔒 **Privacy First** — All processing happens locally by default

## 🚀 Quick Start

### Installation

```bash
# Install via ClawHub (recommended)
clawhub install ying-wen/voice-memo-sync

# Or manually
cd ~/.openclaw/workspace/skills
git clone https://github.com/ying-wen/voice-memo-sync.git
cd voice-memo-sync
./scripts/install.sh
```

### Dependencies

```bash
# Required
brew install ffmpeg

# Optional (enhanced features)
brew install openai-whisper    # Local transcription
brew install yt-dlp            # YouTube/Bilibili download
brew install steipete/tap/remindctl   # Reminders integration
brew install steipete/tap/summarize   # YouTube transcript extraction
```

## 📖 Usage

Just tell your OpenClaw agent:

```
"Sync my voice memos"
"Process this recording: [file]"
"Organize this video: [YouTube/Bilibili URL]"
"Transcribe and summarize this audio"
```

Or use the scripts directly:

```bash
# Process any input
./scripts/process.sh /path/to/audio.mp3
./scripts/process.sh "https://www.youtube.com/watch?v=..."
./scripts/process.sh /path/to/transcript.txt

# Sync iCloud recordings
./scripts/sync-icloud-recordings.sh
```

## 📁 Supported Formats

| Type | Formats | Processing |
|------|---------|------------|
| Voice Memos | .qta, .m4a | Apple native → Whisper fallback |
| Audio | .mp3, .wav, .aac, .flac | Whisper local transcription |
| Video | .mp4, .mov, .mkv, .webm | ffmpeg extract → Whisper |
| YouTube | URL | summarize CLI → yt-dlp fallback |
| Bilibili | URL | yt-dlp download → Whisper |
| Text | .txt, .md | Direct read |
| Documents | .doc, .docx | textutil convert |
| Structured | .json, .csv | Parse and extract |

## 🔧 Configuration

Edit `~/.openclaw/workspace/config/voice-memo-sync.yaml`:

```yaml
sources:
  voice_memos:
    enabled: true
  icloud:
    enabled: true
    paths:
      - "~/Library/Mobile Documents/com~apple~CloudDocs/Recordings"

transcription:
  priority: ["apple", "text", "summarize", "whisper-local"]
  whisper_model: "small"
  language: "auto"

notes:
  folder: "Voice Memos"
  include_quotes: true
  include_original: true

reminders:
  enabled: true
  list: "Reminders"
```

## 📝 Output Format

Notes are created with this structure:

```
🎙️ [Auto-generated Title]

📅 Date | ⏱️ Duration | 👤 Source
🏷️ #tag1 #tag2 #tag3

━━━━━━━━━━━━━━━━━━━━━━

📌 Summary
[One paragraph core summary]

🎯 Key Points
• Point 1
• Point 2
• Point 3

💡 Analysis & Insights
[Personalized analysis based on user context]

📋 Action Items
☐ TODO 1 (synced to Reminders)
☐ TODO 2

💬 Notable Quotes
• "Quote 1"
• "Quote 2"

━━━━━━━━━━━━━━━━━━━━━━

📝 Original Transcript (Cleaned)
[Full transcript text, cleaned up from spoken language]
```

## 🔒 Privacy

- All transcription runs locally by default
- Apple native transcripts extracted from local files
- Whisper runs entirely on your machine
- No data sent to external servers unless you explicitly configure external APIs
- All data stored in local `memory/voice-memos/` directory

## 📂 Data Structure

```
memory/voice-memos/
├── INDEX.md          # Processing records index
├── sources/          # Original file metadata
├── transcripts/      # Raw transcripts
├── processed/        # LLM processed content
└── synced/           # Sync records
```

## 🛠️ Troubleshooting

### Whisper not found
```bash
brew install openai-whisper
```

### yt-dlp download fails
```bash
brew upgrade yt-dlp
# Or use proxy
export ALL_PROXY=http://127.0.0.1:7890
```

### Apple Notes folder not created
```bash
osascript -e 'tell application "Notes" to tell account "iCloud" to make new folder with properties {name:"Voice Memos"}'
```

## 📜 Changelog

### v1.2.0 (2026-03-08)
- Added unified processing script
- Added YouTube/Bilibili support
- Added .doc/.docx/.json/.csv support
- Bilingual SKILL.md (English/Chinese)
- Improved installation script

### v1.0.0 (2026-03-08)
- Initial release
- Apple Voice Memos transcription
- Apple Notes sync
- Whisper fallback

## 📄 License

MIT License - see [LICENSE](LICENSE) for details.

## 🤝 Contributing

Contributions welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) first.

---

Made with ❤️ for the OpenClaw community
