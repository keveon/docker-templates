# Karakeep

自托管书签管理应用（原名 Hoarder），支持保存链接、笔记、图片和 PDF。具备 AI 自动标签、全文搜索和网页存档功能。

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 生成必需的密钥：
   ```bash
   # 生成 NEXTAUTH_SECRET
   openssl rand -base64 36

   # 生成 MEILI_MASTER_KEY
   openssl rand -base64 36
   ```

3. 修改 `.env` 中的配置，至少填写：
   - `NEXTAUTH_SECRET`
   - `NEXTAUTH_URL`
   - `MEILI_MASTER_KEY`

4. 启动服务：
   ```bash
   docker compose up -d
   ```

## 配置说明

### 必填配置

| 变量 | 说明 |
|------|------|
| NEXTAUTH_SECRET | JWT 签名密钥 |
| NEXTAUTH_URL | 服务访问地址 |
| MEILI_MASTER_KEY | Meilisearch 主密钥 |

### 基础配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| KARAKEEP_VERSION | release | 应用版本 |
| TZ | Asia/Shanghai | 时区 |
| LOG_LEVEL | info | 日志级别 |
| MAX_ASSET_SIZE_MB | 50 | 最大上传文件大小 |
| DISABLE_SIGNUPS | false | 禁止新用户注册 |

### 爬虫配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| CRAWLER_NUM_WORKERS | 1 | 爬虫并发任务数 |
| CRAWLER_STORE_SCREENSHOT | true | 保存网页截图 |
| CRAWLER_FULL_PAGE_SCREENSHOT | false | 全页截图 |
| CRAWLER_FULL_PAGE_ARCHIVE | false | 保存网页本地副本 |
| CRAWLER_ENABLE_ADBLOCKER | true | 启用广告拦截 |
| CRAWLER_JOB_TIMEOUT_SEC | 60 | 爬虫任务超时 |
| CRAWLER_VIDEO_DOWNLOAD | false | 下载视频 |
| CRAWLER_VIDEO_DOWNLOAD_MAX_SIZE | 50 | 视频最大大小（MB） |

## OAuth 认证配置

支持通过 OIDC 协议接入第三方认证（如 Authentik、Keycloak 等）：

```bash
DISABLE_PASSWORD_AUTH=true
OAUTH_PROVIDER_NAME=Authentik
OAUTH_WELLKNOWN_URL=https://auth.example.com/application/o/karakeep/.well-known/openid-configuration
OAUTH_CLIENT_ID=your_client_id
OAUTH_CLIENT_SECRET=your_client_secret
OAUTH_ALLOW_DANGEROUS_EMAIL_ACCOUNT_LINKING=true
```

## AI 功能配置

Karakeep 支持通过 AI 自动为书签添加标签和生成摘要。

### 使用 OpenAI

```bash
OPENAI_API_KEY=sk-xxx
INFERENCE_TEXT_MODEL=gpt-4o-mini
INFERENCE_IMAGE_MODEL=gpt-4o-mini
```

支持兼容 OpenAI API 的第三方服务：

```bash
OPENAI_BASE_URL=https://your-api-endpoint.com/v1
OPENAI_API_KEY=sk-xxx
```

### 使用 Ollama（本地模型）

```bash
OLLAMA_BASE_URL=http://ollama:11434
OLLAMA_KEEP_ALIVE=5m
INFERENCE_TEXT_MODEL=llama3.1
INFERENCE_IMAGE_MODEL=llava
```

### 高级推理配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| EMBEDDING_TEXT_MODEL | text-embedding-3-small | 文本嵌入模型 |
| INFERENCE_LANG | chinese | 推理语言 |
| INFERENCE_CONTEXT_LENGTH | 2048 | 最大输入 token |
| INFERENCE_MAX_OUTPUT_TOKENS | 2048 | 最大输出 token |
| INFERENCE_NUM_WORKERS | 1 | 并发推理任务数 |
| INFERENCE_ENABLE_AUTO_TAGGING | true | 启用自动标签 |
| INFERENCE_ENABLE_AUTO_SUMMARIZATION | true | 启用自动摘要 |
| INFERENCE_OUTPUT_SCHEMA | structured | 输出格式 |

## OCR 配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| OCR_LANGS | eng,chi_sim,chi_tra | OCR 语言 |
| OCR_CONFIDENCE_THRESHOLD | 50 | 置信度阈值 |

## SMTP 邮件配置

用于发送邮箱验证邮件：

```bash
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your_username
SMTP_PASSWORD=your_password
SMTP_FROM=noreply@example.com
EMAIL_VERIFICATION_REQUIRED=true
```

## S3 存储配置

支持将资源存储到 S3 兼容服务（如 MinIO）：

```bash
ASSET_STORE_S3_ENDPOINT=https://s3.example.com
ASSET_STORE_S3_REGION=us-east-1
ASSET_STORE_S3_BUCKET=karakeep
ASSET_STORE_S3_ACCESS_KEY_ID=your_access_key
ASSET_STORE_S3_SECRET_ACCESS_KEY=your_secret_key
ASSET_STORE_S3_FORCE_PATH_STYLE=true  # MinIO 需要
```

## 代理配置

用于爬虫的网络代理：

```bash
CRAWLER_HTTP_PROXY=http://proxy.example.com:8080
CRAWLER_HTTPS_PROXY=http://proxy.example.com:8080
CRAWLER_NO_PROXY=localhost,127.0.0.1
```

## Webhook 配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| WEBHOOK_NUM_WORKERS | 1 | 并发任务数 |
| WEBHOOK_TIMEOUT_SEC | 5 | 请求超时 |
| WEBHOOK_RETRY_TIMES | 3 | 重试次数 |

## 服务架构

本模板包含三个服务：

- **web**: 主应用服务
- **chrome**: 无头浏览器（用于网页截图和存档）
- **meilisearch**: 全文搜索引擎

## 相关链接

- [项目主页](https://github.com/karakeep-app/karakeep)
- [官方文档](https://docs.karakeep.app/)
- [配置参考](https://docs.karakeep.app/configuration/)
