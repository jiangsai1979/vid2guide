# vid2guide

A Claude Code skill that turns YouTube videos into step-by-step tutorials with screenshots.

Type `/vid2guide <YouTube URL>` in Claude Code, and it will automatically download the video, transcribe it, identify operation steps, capture screenshots, and generate a Markdown + PDF tutorial.

## Demo

```
/vid2guide https://www.youtube.com/watch?v=4VPoGSeI2sw
```

Output:
```
2026-02-24/BMad-V6/
├── images/
│   ├── step_01.jpg ~ step_08.jpg
├── steps.json              # Structured step data
├── operation_guide.md      # Markdown tutorial with screenshots
├── operation_guide.pdf     # PDF version (images embedded)
├── video.mp4               # Original video
└── video.srt               # Whisper-generated subtitles
```

## How It Works

```
YouTube URL → yt-dlp Download → Whisper STT → AI Step Analysis → ffmpeg Screenshots → AI Vision Enhancement → Markdown/PDF
```

1. **yt-dlp download** — Downloads YouTube video (720p)
2. **Whisper transcription** — Extracts speech-to-text with timestamps
3. **AI step identification** — Doubao LLM analyzes subtitles to identify steps with confidence scores
4. **Screenshot capture** — ffmpeg grabs frames at each step's key timestamp
5. **AI vision enhancement** — For low-confidence steps, sends screenshots to AI for refinement
6. **Document generation** — Produces Markdown + PDF tutorial with embedded screenshots

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/jiangsai1979/vid2guide.git
cd vid2guide
```

### 2. Install dependencies

```bash
# Recommended: use conda for isolation
conda create -n vid2guide python=3.11 -y
conda activate vid2guide

# Install llvmlite via conda first (pip build may fail)
conda install numba -y

# Install Python packages
pip install -r requirements.txt
```

System dependencies:
- [ffmpeg](https://ffmpeg.org/): `brew install ffmpeg` (macOS) or `apt install ffmpeg` (Linux)

### 3. Configure API Key

```bash
cp .env.example .env
```

Edit `.env` and add your Volcengine ARK API Key:
- Go to https://console.volcengine.com/ark
- Create an API Key
- Paste it into `ARK_API_KEY=`

### 4. Set up Claude Code skill

Copy the skill and command files to your project:

```bash
# From the vid2guide repo root
cp claude-code/commands/vid2guide.md /path/to/your/project/.claude/commands/
cp claude-code/skills/vid2guide.md /path/to/your/project/.claude/skills/
```

## Usage

### As a Claude Code Skill (recommended)

In Claude Code, type:

```
/vid2guide https://www.youtube.com/watch?v=xxx
```

That's it. Claude will handle everything automatically.

### As a CLI tool

```bash
# From a YouTube URL
python vid2guide.py --url "https://www.youtube.com/watch?v=xxx"

# From a local video file
python vid2guide.py --video_path your_video.mp4

# With options
python vid2guide.py --url "URL" --output_dir ./output --web_search
```

### CLI Parameters

| Parameter | Description | Default |
|---|---|---|
| `--url` | YouTube URL (auto-download) | — |
| `--video_path` | Path to local video file | Auto-detect MP4 |
| `--srt_path` | Path to existing SRT file | Auto-generate |
| `--output_dir` | Output directory | Named after video |
| `--whisper_model` | Whisper model: tiny/base/small/medium/large | `base` |
| `--max_vision` | Max AI vision enhancement calls | `4` |
| `--web_search` | Enable web search enrichment | Off |
| `--use_video` | Upload video to AI (expensive) | Off |

## Project Structure

```
vid2guide/
├── vid2guide.py          # Core engine
├── requirements.txt
├── .env.example          # API key template
├── LICENSE
├── claude-code/
│   ├── commands/
│   │   └── vid2guide.md  # /vid2guide slash command
│   └── skills/
│       └── vid2guide.md  # Skill knowledge file
└── README.md
```

## Credits

Built on top of [tech-shrimp/video_2_markdown_doubao](https://github.com/tech-shrimp/video_2_markdown_doubao). Key improvements:

- YouTube download integration (yt-dlp)
- Subtitle-driven mode (much cheaper than video upload)
- Confidence scoring + AI vision enhancement
- PDF output with embedded images
- Web search enrichment (optional)
- Claude Code skill integration

## License

MIT
