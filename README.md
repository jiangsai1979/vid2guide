# vid2guide

Turn any video into a step-by-step tutorial with screenshots, powered by AI.

**vid2guide** takes a video file, transcribes it with Whisper, uses AI to identify operation steps, captures screenshots at key moments, and generates a polished Markdown + PDF tutorial — all in one command.

## How It Works

```
Video → Whisper STT → AI Step Analysis → ffmpeg Screenshots → AI Vision Enhancement → Markdown/PDF
```

1. **Whisper transcription** — Extracts speech-to-text with timestamps (`.srt`)
2. **AI step identification** — Doubao LLM analyzes subtitles to identify operation steps with confidence scores
3. **Screenshot capture** — ffmpeg grabs frames at each step's key timestamp
4. **AI vision enhancement** — For low-confidence steps, sends screenshots to AI for description refinement
5. **Document generation** — Produces structured Markdown tutorial with embedded screenshots
6. **PDF export** — Converts Markdown to PDF with images embedded

## Quick Start

### Prerequisites

- Python 3.11+
- [ffmpeg](https://ffmpeg.org/) installed (`brew install ffmpeg`)
- [Volcengine ARK API Key](https://console.volcengine.com/ark) (for Doubao LLM)

### Installation

```bash
git clone https://github.com/jiangsai1979/vid2guide.git
cd vid2guide
pip install -r requirements.txt
```

### Configuration

```bash
cp .env.example .env
# Edit .env and add your ARK_API_KEY
```

### Usage

```bash
# From a YouTube URL (auto-download + generate tutorial)
python vid2guide.py --url "https://www.youtube.com/watch?v=xxx"

# From a local video file
python vid2guide.py --video_path your_video.mp4

# Use existing subtitles (skip Whisper)
python vid2guide.py --video_path your_video.mp4 --srt_path subtitles.srt

# Enable web search to enrich documentation
python vid2guide.py --url "https://www.youtube.com/watch?v=xxx" --web_search
```

### Parameters

| Parameter | Description | Default |
|---|---|---|
| `--url` | YouTube URL (auto-download with yt-dlp) | — |
| `--video_path` | Path to local video file | Auto-detect MP4 in current dir |
| `--srt_path` | Path to SRT subtitle file | Auto-generate with Whisper |
| `--output_dir` | Output directory | Named after video file |
| `--whisper_model` | Whisper model size | `base` |
| `--max_vision` | Max AI vision enhancement calls | `4` |
| `--web_search` | Enable web search enrichment | Off |
| `--use_video` | Upload video to AI (expensive) | Off |
| `--fps` | Frame extraction rate | `1` |
| `--api_key` | ARK API Key (overrides .env) | From `.env` |

## Output

```
output_dir/
├── images/
│   ├── step_01.jpg
│   ├── step_02.jpg
│   └── ...
├── steps.json            # Structured step data
├── operation_guide.md    # Markdown tutorial
├── operation_guide.pdf   # PDF tutorial (images embedded)
├── video.mp4             # Original video copy
└── video.srt             # Subtitle file
```

## Use as a Claude Code Skill

vid2guide can be used as a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom command. Copy the files from the `claude-code/` directory:

```bash
# From the vid2guide repo root
cp -r claude-code/commands/ /path/to/your/project/.claude/commands/
cp -r claude-code/skills/ /path/to/your/project/.claude/skills/
```

Then in Claude Code, type `/vid2guide https://youtube.com/watch?v=xxx` to auto-download and generate a tutorial.

## Credits

Built on top of [tech-shrimp/video_2_markdown_doubao](https://github.com/tech-shrimp/video_2_markdown_doubao). Key improvements:

- Subtitle-driven mode (much cheaper than video upload)
- Confidence scoring + AI vision enhancement
- PDF output with embedded images
- Web search enrichment (optional)
- Unified output directory management

## License

MIT
