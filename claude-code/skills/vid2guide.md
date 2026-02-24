---
name: vid2guide
description: Use when 用户输入 /vid2guide <YouTube URL>，或要求把 YouTube 视频自动转换为带截图的图文教程。
---

# YouTube 视频转图文教程 (vid2guide)

## 触发条件

当用户输入 `/vid2guide <URL>` 或要求「把这个 YouTube 视频转成图文教程」时激活此 skill。

## 前置依赖

- conda 环境 `vid2guide`（Python 3.11）
- 当前工作区满足以下任一结构：
  - 根目录包含 `vid2guide.py`
  - 根目录包含 `vid2guide/vid2guide.py`
- 对应项目目录下 `.env` 已配置 `ARK_API_KEY`（火山引擎豆包大模型）

## 首次配置

先定位项目目录变量 `PROJECT_DIR`：

```bash
if [ -f "vid2guide.py" ]; then
  PROJECT_DIR="."
elif [ -f "vid2guide/vid2guide.py" ]; then
  PROJECT_DIR="vid2guide"
else
  echo "未找到 vid2guide.py，请先克隆/放置 vid2guide 项目"
  exit 1
fi
```

如果 `$PROJECT_DIR/.env` 不存在，执行以下步骤引导用户配置：

1. 复制模板：`cp "$PROJECT_DIR/.env.example" "$PROJECT_DIR/.env"`
2. 提示用户填写 API Key：
   - 打开 https://console.volcengine.com/ark 创建 API Key
   - 将 key 填入 `$PROJECT_DIR/.env` 的 `ARK_API_KEY=` 后面
3. 确认 `.env` 中 `ARK_API_KEY` 不为空后再继续执行

## 执行流程

收到 YouTube URL 后，严格按以下步骤自动执行，不要询问用户确认：

### 步骤 1：定位项目目录并创建输出目录

```bash
workspace_root=$(pwd)

if [ -f "vid2guide.py" ]; then
  PROJECT_DIR="."
elif [ -f "vid2guide/vid2guide.py" ]; then
  PROJECT_DIR="vid2guide"
else
  echo "未找到 vid2guide.py，请先克隆/放置 vid2guide 项目"
  exit 1
fi

date_dir=$(date +%Y-%m-%d)
output_name="guide_$(date +%H%M%S)"
mkdir -p "$workspace_root/$date_dir/$output_name"
```

### 步骤 2：运行 vid2guide（自动下载 + 生成教程）

在 `$PROJECT_DIR` 目录下执行：

```bash
(
  cd "$PROJECT_DIR" || exit 1
  eval "$(conda shell.bash hook)" && conda activate vid2guide && python vid2guide.py \
    --url "YOUTUBE_URL" \
    --output_dir "$workspace_root/$date_dir/$output_name"
)
```

参数说明：
- `--url`：YouTube URL，自动下载视频
- `--output_dir`：输出目录（建议使用绝对路径，避免目录层级差异）
- `--whisper_model`：默认 base，可选 tiny/small/medium/large
- `--max_vision 4`：AI 看图增强次数
- `--web_search`：启用联网搜索增强文档

### 步骤 3：确认产物并报告

检查 `$workspace_root/$date_dir/$output_name` 包含以下文件后向用户报告：
- `operation_guide.md` — Markdown 图文教程
- `operation_guide.pdf` — PDF 版本
- `images/step_XX.jpg` — 步骤截图
- `steps.json` — 结构化步骤数据

## 错误处理

| 错误 | 解决 |
|------|------|
| yt-dlp 下载失败 | `pip install --upgrade yt-dlp` |
| Whisper 模型下载慢 | 首次运行需等待，后续有缓存 |
| ARK API 404 | 检查 `.env` 中 API Key，模型名应为 `doubao-seed-2-0-pro-260215` |
| ffmpeg 未安装 | `brew install ffmpeg` |
| 截图生成不完整（部分失败） | 检查日志中的 WARNING 信息，通常是 ffmpeg 不支持视频编码格式（如 AV1） |
| 所有截图均生成失败（RuntimeError） | 确认 ffmpeg 支持该视频编码；可尝试安装完整版 ffmpeg: `brew install ffmpeg` |
| conda 环境不存在 | `conda create -n vid2guide python=3.11 && conda activate vid2guide && pip install -r "$PROJECT_DIR/requirements.txt"` |

## 关键文件

- API Key：`$PROJECT_DIR/.env`
- 模型配置：`$PROJECT_DIR/vid2guide.py` 第 38 行 `self.model`
- Whisper 模型：`$PROJECT_DIR/.env` 中 `WHISPER_MODEL=base`
