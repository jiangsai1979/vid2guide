请将以下 YouTube 视频下载并转为图文教程，完整自动执行，不要询问确认：

YouTube URL: $ARGUMENTS

## 执行步骤

0. 先定位项目目录（支持两种结构）：
   - 若当前目录存在 `vid2guide.py`，则 `PROJECT_DIR=.`
   - 若当前目录存在 `vid2guide/vid2guide.py`，则 `PROJECT_DIR=vid2guide`
   - 若都不存在，提示用户先克隆/放置 vid2guide 项目后结束
1. 检查 `$PROJECT_DIR/.env` 是否存在且 ARK_API_KEY 已配置。如果不存在，先执行 `cp "$PROJECT_DIR/.env.example" "$PROJECT_DIR/.env"`，然后提示用户去 https://console.volcengine.com/ark 获取 API Key 并填入 `.env` 文件
2. 创建今天日期目录（格式 `YYYY-MM-DD`）以及本次输出目录（如 `guide_HHMMSS`）
3. 在 `$PROJECT_DIR` 目录下运行：
   ```
   eval "$(conda shell.bash hook)" && conda activate vid2guide && python vid2guide.py --url "URL" --output_dir "工作区绝对路径/YYYY-MM-DD/guide_HHMMSS"
   ```
4. 确认产物（`operation_guide.md`、`operation_guide.pdf`、截图、`steps.json`）后报告完成

## 错误处理
- yt-dlp 失败 → `pip install --upgrade yt-dlp` 后重试
- ARK API 404 → 检查 `$PROJECT_DIR/.env` 中 API Key，模型名 doubao-seed-2-0-pro-260215
- 截图全部失败（RuntimeError） → ffmpeg 不支持视频编码，`brew install ffmpeg` 安装完整版
- conda 环境不存在 → `conda create -n vid2guide python=3.11 && conda activate vid2guide && pip install -r "$PROJECT_DIR/requirements.txt"`
