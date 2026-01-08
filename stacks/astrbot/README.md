# AstrBot

一站式多平台大模型聊天机器人平台，支持 QQ、Telegram、企业微信、飞书、钉钉等多种平台接入，内置 Agent 能力、知识库和丰富的插件生态。

## 功能特性

- 🤖 **多平台支持** - QQ、Telegram、企业微信、飞书、钉钉、Discord、Slack 等
- 🧠 **大模型集成** - 支持 OpenAI、Gemini、Claude、DeepSeek、GLM、Ollama 等
- 🎯 **Agent 能力** - 内置智能 Agent，支持复杂任务执行
- 📚 **知识库** - 支持 PDF、DOCX、Markdown 等格式文档上传，混合检索
- 🔌 **插件生态** - 640+ 社区插件，轻松扩展功能
- 🎨 **WebUI 管理** - 图形化配置界面，无需修改配置文件
- 🐳 **Docker 部署** - 支持一键部署，开箱即用

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. （可选）修改 `.env` 中的配置

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 配置反向代理后访问 WebUI（默认端口 6185）

5. 首次访问需初始化配置：
   - 选择消息平台（如 QQ、Telegram）
   - 配置 LLM API Key（推荐使用 OpenAI 或 DeepSeek）
   - 创建会话开始使用

## 平台接入配置

### QQ 平台

推荐使用以下协议之一：

1. **NapCat**（推荐）
   - 安装 NapCat 并配置 QQ 账号
   - 在 AstrBot WebUI 中添加 NapCat 适配器
   - 配置 WebSocket 地址

2. **LLOneBot**
   - 使用 NTQQ 安装 LLOneBot 插件
   - 配置正向 WebSocket 连接

3. **ShardChannel**
   - 适用于 QQ 频道场景

### Telegram Bot

1. 在 Telegram 中通过 [@BotFather](https://t.me/BotFather) 创建 Bot
2. 获取 Bot Token
3. 在 AstrBot WebUI 中添加 Telegram 适配器
4. 填入 Token 即可

### 企业微信

支持企业微信应用、智能机器人、微信客服、微信公众号等多种接入方式，详见[官方文档](https://astrbot.app/documentation)。

## LLM 配置

### OpenAI（推荐）

```bash
# 在 WebUI 中配置
提供商: OpenAI
API Key: sk-xxx
Base URL: https://api.openai.com/v1
模型: gpt-4o-mini
```

### DeepSeek（性价比推荐）

```bash
# 在 WebUI 中配置
提供商: OpenAI Compatible
API Key: sk-xxx
Base URL: https://api.deepseek.com/v1
模型: deepseek-chat
```

### Ollama（本地部署）

```bash
# 在 WebUI 中配置
提供商: OpenAI Compatible
API Key: ollama
Base URL: http://host.docker.internal:11434/v1
模型: llama3.2
```

> 💡 **提示**：更多 LLM 提供商配置请参考 [官方文档](https://astrbot.app/documentation)

## 插件管理

AstrBot 拥有活跃的社区生态，提供 640+ 插件：

### 安装插件

1. 访问 WebUI → 插件市场
2. 浏览或搜索需要的插件
3. 点击安装即可自动部署

### 推荐插件

- **心流** - 基于心流机制的群聊主动回复
- **表情包发送** - AI 智能发送表情，支持云端同步
- **pixiv_search** - 搜索 Pixiv 插画、小说
- **LivingMemory** - 长期记忆插件，模拟人类记忆全流程

更多插件请访问 [插件展示页](https://astrbot.app/plugins)

## 数据持久化

所有数据（配置文件、插件、知识库、聊天记录等）存储在 Docker 命名卷 `astrbot_astrbot_data` 中：

```bash
# 查看卷位置
docker volume inspect astrbot_astrbot_data

# 备份数据
docker run --rm \
  -v astrbot_astrbot_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/astrbot-backup-$(date +%Y%m%d).tar.gz -C /data .

# 恢复数据
docker run --rm \
  -v astrbot_astrbot_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/astrbot-backup-YYYYMMDD.tar.gz -C /data
```

## 反向代理配置

服务默认在 Docker 网络内部暴露 6185 端口（WebUI），需通过反向代理访问。

### Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name bot.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://astrbot:6185;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Caddy 配置示例

```caddyfile
bot.example.com {
    reverse_proxy astrbot:6185
}
```

## 端口说明

| 端口 | 用途 | 必需 |
|------|------|------|
| 6185 | WebUI 管理界面 | 是 |
| 6199 | QQ WebSocket 连接 | 可选 |
| 6195 | 企业微信 Webhook | 可选 |
| 6196 | QQ 官方接口 Webhook | 可选 |

> ⚠️ **注意**：所有端口默认仅在 Docker 网络内部暴露，如需外部访问请通过反向代理或修改 [compose.yaml](compose.yaml) 中的 `expose` 为 `ports`。

## 系统要求

- **最小配置**：1 vCPU、1GB RAM、5GB 磁盘
- **生产推荐**：2+ vCPU、2-4GB RAM、SSD 20GB+
- **网络要求**：需访问 LLM API（如 OpenAI、DeepSeek）

## 注意事项

1. **首次启动**：容器首次启动需要初始化，可能需要 30-60 秒，请耐心等待

2. **LLM API**：需自行申请 LLM API Key，推荐使用 DeepSeek（高性价比）或 OpenAI

3. **QQ 接入**：QQ 平台需要第三方协议（如 NapCat），请确保协议正常运行

4. **插件安全**：安装社区插件时请注意安全，建议优先使用官方认证插件

5. **数据备份**：定期备份 `astrbot_astrbot_data` 卷，避免配置丢失

## 常见问题

### Q: WebUI 无法访问？

A: 检查容器是否正常运行：
```bash
docker compose ps
docker compose logs astrbot
```

### Q: QQ 消息无法接收？

A: 确认以下几点：
1. NapCat/LLOneBot 是否正常运行
2. WebSocket 配置是否正确
3. 防火墙是否放行相关端口

### Q: LLM 响应慢或超时？

A: 可能原因：
1. API 网络问题（建议使用代理）
2. 模型选择过大（推荐使用 `gpt-4o-mini` 或 `deepseek-chat`）
3. Token 超限（在 WebUI 中调整上下文长度）

### Q: 如何更新到最新版本？

A: 拉取最新镜像并重启：
```bash
docker compose pull
docker compose up -d
```

## 相关链接

- [GitHub 仓库](https://github.com/AstrBotDevs/AstrBot)
- [官方文档](https://astrbot.app)
- [插件市场](https://astrbot.app/plugins)
- [社区论坛](https://github.com/AstrBotDevs/AstrBot/discussions)
