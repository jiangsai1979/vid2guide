请将以下 YouTube 视频下载并转为图文教程，完整自动执行，不要询问确认：

YouTube URL: $ARGUMENTS

## 执行步骤

0. 检查 vid2guide/.env 是否存在且 ARK_API_KEY 已配置。如果不存在，先执行 `cp vid2guide/.env.example vid2guide/.env`，然后提示用户去 https://console.volcengine.com/ark 获取 API Key 并填入 .env 文件
1. 创建今天日期目录（格式 YYYY-MM-DD）
2. 在 vid2guide/ 目录下运行：
   ```
   python vid2guide.py --url "URL" --output_dir "../YYYY-MM-DD/输出目录名"
   ```
3. 确认产物（operation_guide.md、PDF、截图、steps.json）后报告完成

## 错误处理
- yt-dlp 失败 → `pip install --upgrade yt-dlp` 后重试
- ARK API 404 → 检查 vid2guide/.env 中 API Key，模型名 doubao-seed-2-0-pro-260215
- conda 环境不存在 → `conda create -n vid2guide python=3.11 && conda activate vid2guide && pip install -r vid2guide/requirements.txt`
