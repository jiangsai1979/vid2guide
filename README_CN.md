# vid2guide

一个 Claude Code Skill，将 YouTube 视频一键转为带截图的步骤教程。

在 Claude Code 中输入 `/vid2guide <YouTube URL>`，自动完成：下载视频 → Whisper 转录 → AI 识别步骤 → 截图 → 生成 Markdown + PDF 教程。

## 效果演示

```
/vid2guide https://www.youtube.com/watch?v=4VPoGSeI2sw
```

输出：
```
2026-02-24/BMad-V6/
├── images/
│   ├── step_01.jpg ~ step_08.jpg
├── steps.json              # 结构化步骤数据
├── operation_guide.md      # Markdown 图文教程
├── operation_guide.pdf     # PDF 版本（图片内嵌）
├── video.mp4               # 原始视频
└── video.srt               # Whisper 生成的字幕
```

## 工作原理

```
YouTube URL → yt-dlp 下载 → Whisper 语音转文字 → AI 步骤识别 → ffmpeg 截图 → AI 看图增强 → Markdown/PDF
```

1. **yt-dlp 下载** — 下载 YouTube 视频（720p）
2. **Whisper 转录** — 语音转文字，生成带时间戳的字幕
3. **AI 步骤识别** — 豆包大模型分析字幕，识别操作步骤并给出置信度评分
4. **截图生成** — ffmpeg 在每个步骤的关键时间点截取画面
5. **AI 看图增强** — 对低置信度步骤，发送截图给 AI 修正描述
6. **文档生成** — 生成带截图的 Markdown + PDF 教程

## 安装

### 1. 克隆仓库

```bash
git clone https://github.com/jiangsai1979/vid2guide.git
cd vid2guide
```

### 2. 安装依赖

```bash
# 推荐使用 conda 隔离环境
conda create -n vid2guide python=3.11 -y
conda activate vid2guide

# 先用 conda 安装 llvmlite（pip 编译可能失败）
conda install numba -y

# 安装 Python 包
pip install -r requirements.txt
```

系统依赖：
- [ffmpeg](https://ffmpeg.org/)：`brew install ffmpeg`（macOS）或 `apt install ffmpeg`（Linux）

### 3. 配置 API Key

```bash
cp .env.example .env
```

编辑 `.env`，填入火山引擎 ARK API Key：
- 访问 https://console.volcengine.com/ark
- 创建 API Key
- 粘贴到 `ARK_API_KEY=` 后面

### 4. 配置 Claude Code Skill

将 skill 和 command 文件复制到你的项目：

```bash
cp claude-code/commands/vid2guide.md /你的项目/.claude/commands/
cp claude-code/skills/vid2guide.md /你的项目/.claude/skills/
```

## 使用方式

### 作为 Claude Code Skill（推荐）

在 Claude Code 中输入：

```
/vid2guide https://www.youtube.com/watch?v=xxx
```

就这样，Claude 会自动处理一切。

### 作为命令行工具

```bash
# 从 YouTube URL
python vid2guide.py --url "https://www.youtube.com/watch?v=xxx"

# 从本地视频
python vid2guide.py --video_path your_video.mp4

# 带选项
python vid2guide.py --url "URL" --output_dir ./output --web_search
```

### 命令行参数

| 参数 | 说明 | 默认值 |
|---|---|---|
| `--url` | YouTube URL（自动下载） | — |
| `--video_path` | 本地视频文件路径 | 自动查找 MP4 |
| `--srt_path` | 已有字幕文件路径 | 自动生成 |
| `--output_dir` | 输出目录 | 以视频名命名 |
| `--whisper_model` | Whisper 模型：tiny/base/small/medium/large | `base` |
| `--max_vision` | AI 看图增强最大次数 | `4` |
| `--web_search` | 启用联网搜索增强文档 | 关闭 |
| `--use_video` | 上传视频给 AI 看（较贵） | 关闭 |

## 项目结构

```
vid2guide/
├── vid2guide.py          # 核心引擎
├── requirements.txt
├── .env.example          # API Key 模板
├── LICENSE
├── claude-code/
│   ├── commands/
│   │   └── vid2guide.md  # /vid2guide 斜杠命令
│   └── skills/
│       └── vid2guide.md  # Skill 知识文件
├── README.md             # English
└── README_CN.md          # 中文
```

## 致谢

基于 [tech-shrimp/video_2_markdown_doubao](https://github.com/tech-shrimp/video_2_markdown_doubao) 改造。主要改进：

- 集成 YouTube 下载（yt-dlp）
- 字幕驱动模式（比视频上传便宜得多）
- 置信度评分 + AI 看图增强
- PDF 输出（图片内嵌）
- 联网搜索增强（可选）
- Claude Code Skill 集成

## 许可证

MIT
