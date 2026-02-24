---
name: vid2guide
description: YouTube 视频一键转图文教程。下载视频 → Whisper 转录 → AI 步骤识别 → 截图 → 生成 Markdown/PDF 文档。使用方式：/vid2guide <YouTube URL>
---

# YouTube 视频转图文教程 (vid2guide)

## 触发条件

当用户输入 `/vid2guide <URL>` 或要求「把这个 YouTube 视频转成图文教程」时激活此 skill。

## 前置依赖

- conda 环境 `vid2guide`（Python 3.11）
- `vid2guide/` 目录（核心代码）
- `vid2guide/.env` 已配置 `ARK_API_KEY`（火山引擎豆包大模型）

## 首次配置

如果 `vid2guide/.env` 不存在，执行以下步骤引导用户配置：

1. 复制模板：`cp vid2guide/.env.example vid2guide/.env`
2. 提示用户填写 API Key：
   - 打开 https://console.volcengine.com/ark 创建 API Key
   - 将 key 填入 `vid2guide/.env` 的 `ARK_API_KEY=` 后面
3. 确认 `.env` 中 `ARK_API_KEY` 不为空后再继续执行

## 执行流程

收到 YouTube URL 后，严格按以下步骤自动执行，不要询问用户确认：

### 步骤 1：创建输出目录

```bash
date_dir=$(date +%Y-%m-%d)
mkdir -p "$date_dir"
```

### 步骤 2：运行 vid2guide（自动下载 + 生成教程）

在 `vid2guide/` 目录下执行：

```bash
eval "$(conda shell.bash hook)" && conda activate vid2guide && python vid2guide.py \
  --url "YOUTUBE_URL" \
  --output_dir "../$date_dir/输出目录名"
```

参数说明：
- `--url`：YouTube URL，自动下载视频
- `--output_dir`：输出目录，用视频标题的简短英文版命名
- `--whisper_model`：默认 base，可选 tiny/small/medium/large
- `--max_vision 4`：AI 看图增强次数
- `--web_search`：启用联网搜索增强文档

### 步骤 3：确认产物并报告

检查输出目录包含以下文件后向用户报告：
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
| conda 环境不存在 | `conda create -n vid2guide python=3.11 && conda activate vid2guide && pip install -r requirements.txt` |

## 关键文件

- API Key：`vid2guide/.env`
- 模型配置：`vid2guide/vid2guide.py` 第 38 行 `self.model`
- Whisper 模型：`vid2guide/.env` 中 `WHISPER_MODEL=base`
