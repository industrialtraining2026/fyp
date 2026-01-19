# Railway 部署指南

## 概述
这个文档说明如何将 FYP 应用部署到 Railway 平台。

## Railway 免费方案
- **试用期**: 30 天 + $5 使用额度
- **需要**: GitHub 账号（你已经有了）
- **不需要**: 信用卡（试用期）
- **试用期后**: 自动转为 Free 计划（每月 $1 额度，资源有限）

## 部署前准备

### 1. 确保代码已推送到 GitHub
- 将所有代码推送到 GitHub 仓库
- Railway 会从 GitHub 部署

### 2. 需要修改的代码

#### A. 修改后端：让 FastAPI 服务前端静态文件

需要在 `server/main.py` 中添加静态文件服务。

**修改位置**: 在 `app = FastAPI(...)` 之后，添加静态文件 mount

**需要添加的代码**:
```python
from fastapi.staticfiles import StaticFiles
from fastapi.responses import FileResponse

# 在 app = FastAPI(...) 之后添加
# Mount static files (frontend)
static_path = Path(__file__).parent.parent / "web"
app.mount("/static", StaticFiles(directory=str(static_path)), name="static")

# Serve index.html at root
@app.get("/")
async def read_root():
    return FileResponse(str(static_path / "index.html"))

# Serve other HTML files
@app.get("/{path:path}")
async def serve_frontend(path: str):
    file_path = static_path / path
    if file_path.exists() and file_path.is_file():
        return FileResponse(str(file_path))
    # If it's an API route, let FastAPI handle it
    raise HTTPException(status_code=404, detail="Not found")
```

#### B. 修改前端 API 地址

需要将所有前端文件中的硬编码 API 地址改为相对路径。

**需要修改的文件**:
1. `web/app.js` - 第8行
2. `web/login.html` - 第218行  
3. `web/submission.html` - 第326行
4. `web/notification.js` - 第1行
5. `web/teacher_dashboard.js` - 第1行

**修改方式**: 
将所有 `const API_BASE = 'http://127.0.0.1:8000';`
改为: `const API_BASE = '';` 或 `const API_BASE = window.location.origin;`

### 3. 环境变量配置

在 Railway Dashboard 中设置以下环境变量：

- `OPENAI_API_KEY` (可选，如果你使用 OpenAI)
- `GOOGLE_API_KEY` (可选，如果你使用 Google AI)
- `GROQ_API_KEY` (可选，如果你使用 Groq)
- `PORT` (Railway 会自动设置，不需要手动配置)
- `HOST` (Railway 会自动处理，不需要配置)

## 部署步骤

### 1. 注册 Railway 账号
1. 访问 https://railway.app
2. 使用 GitHub 账号登录
3. 验证 GitHub 账号（不需要信用卡）

### 2. 创建新项目
1. 点击 "New Project"
2. 选择 "Deploy from GitHub repo"
3. 选择你的 GitHub 仓库
4. Railway 会自动检测 Python 项目

### 3. 配置环境变量
1. 在项目设置中找到 "Variables"
2. 添加需要的环境变量（API keys 等）

### 4. 部署
1. Railway 会自动开始部署
2. 等待构建完成（通常 2-5 分钟）
3. 查看日志确认部署成功

### 5. 获取公开 URL
1. 在项目设置中找到 "Networking"
2. 点击 "Generate Domain" 生成公开 URL
3. 或者使用默认提供的 URL

## 数据持久化

Railway 免费试用期提供 0.5GB 持久化存储（Volume）。

**注意**: 
- 试用期结束后，Free 计划的存储限制更严格
- 如果需要更多存储，可能需要升级到 Hobby 计划（$5/月）

## 监控和使用量

在 Railway Dashboard 可以：
- 查看资源使用量（RAM, CPU, 存储）
- 查看日志
- 监控 $5 credit 的使用情况

## 故障排除

### 应用无法启动
- 检查日志中的错误信息
- 确认所有环境变量已设置
- 确认 `requirements.txt` 包含所有依赖

### 前端无法访问
- 确认已修改代码让 FastAPI 服务静态文件
- 确认 API_BASE 已改为相对路径
- 检查 Railway 日志

### 数据丢失
- 确认已配置 Volume（持久化存储）
- 试用期结束后，Free 计划的数据可能不稳定

## 下一步

部署成功后：
1. 测试所有功能（登录、上传 PDF、使用 chatbot 等）
2. 分享公开 URL 给老师和学生
3. 监控使用量和费用

如果 30 天后需要继续使用：
- 考虑升级到 Hobby 计划（$5/月）
- 或者迁移到其他平台（如 Fly.io）

