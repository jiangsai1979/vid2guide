---
name: vid2guide
description: Use when a user asks to convert a YouTube video into a step-by-step tutorial with screenshots and Markdown/PDF output.
---

# vid2guide

Convert a YouTube video into a tutorial package: download video, transcribe with Whisper, detect steps, capture screenshots, and generate Markdown/PDF.

## When to use

- User asks for "video to tutorial", "YouTube to guide", or equivalent.
- Input is a YouTube URL and expected output is a structured, screenshot-based guide.

## Preconditions

- Conda env `vid2guide` (Python 3.11) is available.
- Workspace has one of these layouts:
  - `./vid2guide.py`
  - `./vid2guide/vid2guide.py`
- `ARK_API_KEY` is set in `.env` under the detected project directory.

## Workflow

1. Resolve project directory:

```bash
workspace_root=$(pwd)
if [ -f "vid2guide.py" ]; then
  PROJECT_DIR="."
elif [ -f "vid2guide/vid2guide.py" ]; then
  PROJECT_DIR="vid2guide"
else
  echo "vid2guide.py not found; place/clone vid2guide project first"
  exit 1
fi
```

2. Ensure env file exists:

```bash
if [ ! -f "$PROJECT_DIR/.env" ]; then
  cp "$PROJECT_DIR/.env.example" "$PROJECT_DIR/.env"
  echo "Please fill ARK_API_KEY in $PROJECT_DIR/.env, then rerun."
  exit 1
fi
```

3. Create output directory and run:

```bash
date_dir=$(date +%Y-%m-%d)
output_name="guide_$(date +%H%M%S)"
output_dir="$workspace_root/$date_dir/$output_name"
mkdir -p "$output_dir"

(
  cd "$PROJECT_DIR" || exit 1
  eval "$(conda shell.bash hook)" && conda activate vid2guide && python vid2guide.py \
    --url "YOUTUBE_URL" \
    --output_dir "$output_dir"
)
```

4. Verify artifacts in `$output_dir`:

- `operation_guide.md`
- `operation_guide.pdf`
- `images/step_XX.jpg`
- `steps.json`

## Common fixes

- `yt-dlp` failure: `pip install --upgrade yt-dlp`
- ARK 404: check `ARK_API_KEY` and model `doubao-seed-2-0-pro-260215`
- ffmpeg missing/codec issue: install full ffmpeg (`brew install ffmpeg`)
- conda env missing:
  `conda create -n vid2guide python=3.11 && conda activate vid2guide && pip install -r "$PROJECT_DIR/requirements.txt"`
