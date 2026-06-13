# Video Summarizer

A Claude Code skill that downloads videos from YouTube, Bilibili, Twitter/X, TikTok and 1800+ platforms, then generates AI-powered summaries with transcripts.

> **Note**: This is a skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI. Not affiliated with Anthropic.

## Quick Start

### Method 1: Claude Code Plugin Market (Recommended)

```bash
# Add marketplace and install
/plugin marketplace add liang121/video-summarizer
/plugin install video-summarizer@video-summarizer
```

### Method 2: Manual Installation

```bash
# Clone repository
git clone https://github.com/liang121/video-summarizer.git ~/.claude/plugins/video-summarizer

# Add plugin in Claude Code
/plugin add ~/.claude/plugins/video-summarizer
```

### 2. Use the Skill

**Quick Usage:**
```bash
/video-summarizer <video-url>
```

**Or use natural language:**
```
Summarize this video: <video-url>
Download this video: https://www.bilibili.com/video/BVxxxxx
```

The skill will automatically:
- **Auto-install dependencies** (yt-dlp, ffmpeg, faster-whisper) if needed
- Download the video and extract subtitles
- Transcribe with Whisper if no subtitles available
- Generate an AI summary

## Features

- **Multi-platform support** - YouTube, Bilibili, Twitter/X, Vimeo, TikTok, Instagram, Twitch, and [1800+ more sites](https://github.com/yt-dlp/yt-dlp/blob/master/supportedsites.md)
- **Auto subtitle extraction** - Downloads manual or auto-generated subtitles when available
- **Parallel Whisper transcription** - Uses faster-whisper with silence-based splitting for 4-5x faster speech-to-text
- **AI-powered summaries** - Generates structured summaries with key points, quotes, and timestamps
- **Complete resource package** - Outputs video (MP4), audio (MP3), subtitles (VTT), transcript (TXT), and summary (MD)
- **Auto dependency management** - Claude Code installs everything for you

## Usage

### Slash Command (Direct)

```bash
/video-summarizer <video-url>
```

**Examples:**
```bash
/video-summarizer https://www.youtube.com/watch?v=xxxxx
/video-summarizer https://www.bilibili.com/video/BVxxxxx
/video-summarizer https://twitter.com/user/status/xxxxx
```

### Natural Language (Auto-trigger)

The skill also activates automatically when you say:

- "Summarize this video: [URL]"
- "Download this video: [URL]"
- "What's in this video? [URL]"
- "Transcribe this video: [URL]"

### Example

```
Summarize this video: <video-url>
```

### Output

All files are saved to `./downloads/<video-title>/`:

```
downloads/
└── Video_Title/
    ├── video.mp4          # Original video (up to 1080p)
    ├── audio.mp3          # Extracted audio
    ├── subtitle.vtt       # Subtitles with timestamps
    ├── transcript.txt     # Plain text transcript
    └── summary.md         # AI-generated summary
```

## Parallel Transcription

For videos without subtitles, the skill uses `parallel_transcribe.py` which:

- **Auto-splits long audio** at silence points (avoids cutting mid-speech)
- **Parallel processing** using multiple CPU cores
- **4-5x faster** than standard openai-whisper

### Usage

```bash
python scripts/parallel_transcribe.py \
  --input audio.mp3 \
  --output-dir ./output \
  --model small \
  --language auto
```

### Options

| Option | Default | Description |
|--------|---------|-------------|
| `--model` | small | tiny/base/small/medium/large-v3 |
| `--language` | auto | Language code or 'auto' for detection |
| `--workers` | CPU/2 | Number of parallel workers |
| `--min-segment` | 60 | Min duration (sec) to enable splitting |

### Performance

| Audio Length | Single Process | Parallel (4 workers) |
|--------------|----------------|----------------------|
| 5 min | ~2-3 min | ~30-60 sec |
| 30 min | ~15-20 min | ~3-5 min |
| 1 hour | ~40+ min | ~8-12 min |

*Based on small model, CPU processing*

## Configuration

### Customize Summary Format

Edit `reference/summary-prompt.md` to customize the output. Available variables:

| Variable | Description |
|----------|-------------|
| `{{TITLE}}` | Video title |
| `{{PLATFORM}}` | Platform name |
| `{{URL}}` | Original URL |
| `{{DURATION}}` | Video duration |
| `{{TRANSCRIPT}}` | Full transcript |

### Whisper Models

| Model | Size | Speed | Quality |
|-------|------|-------|---------|
| tiny | 39M | Fastest | Basic |
| base | 74M | Fast | Good |
| **small** | 244M | Medium | **Recommended** |
| medium | 769M | Slow | Better |
| large-v3 | 1.5G | Slowest | Best |

## Platform Notes

### YouTube
- Supports manual and auto-generated subtitles
- Premium content requires browser cookies

### Bilibili
- Prioritizes Chinese subtitles
- Member content requires: `--cookies-from-browser chrome`

### Twitter/X
- Usually requires Whisper (no native subtitles)
- This skill downloads and summarizes media from a known URL. For tweet search,
  reply search, user lookup, follower export, media upload, direct messages,
  monitors, webhooks, giveaway draws, posting, or OpenClaw plugin workflows, use
  a separate approved companion such as
  [TweetClaw](https://github.com/Xquik-dev/tweetclaw) and keep its outputs as
  source evidence unless you separately approve an action.

### Authenticated Content
```bash
yt-dlp --cookies-from-browser chrome "URL"
```

Treat browser cookies as account credentials. Use `--cookies-from-browser` only
on a trusted machine, and do not paste exported cookies into prompts, commits,
issue text, or shared logs.

## Manual Installation (Optional)

If you prefer to install dependencies yourself instead of letting Claude Code handle it:

```bash
bash ~/.claude/plugins/video-summarizer/skills/video-summarizer/scripts/install_deps.sh
```

This script installs:
- **yt-dlp** - Video downloading
- **ffmpeg** - Audio/video processing
- **faster-whisper** - Fast speech-to-text

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No subtitles | Whisper will auto-transcribe |
| Download fails | Try `--cookies-from-browser chrome` |
| Platform not supported | Check: `yt-dlp --list-extractors \| grep -i "name"` |
| Missing dependencies | Ask Claude Code to "install video-summarizer dependencies" |

## License

MIT License - see [LICENSE](./LICENSE)

## Credits

- [yt-dlp](https://github.com/yt-dlp/yt-dlp) - Video downloading
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper) - Fast speech recognition
- [FFmpeg](https://ffmpeg.org/) - Media processing
