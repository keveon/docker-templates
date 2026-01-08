# Octopus

LLM API 聚合与负载均衡服务，支持多渠道聚合、智能负载均衡、协议转换、价格同步等功能。

## 功能特性

- 🔀 **多渠道聚合** - 统一管理多个 LLM 提供商渠道
- ⚖️ **负载均衡** - 支持轮询、随机、故障转移、加权等策略
- 🔄 **协议转换** - OpenAI Chat / OpenAI Responses / Anthropic / Gemini 协议互转
- 💰 **价格同步** - 自动同步模型定价数据
- 🔃 **模型同步** - 自动同步可用模型列表
- 📊 **统计分析** - 请求统计、Token 消耗、成本追踪
- 🗄️ **多数据库支持** - SQLite、MySQL、PostgreSQL

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 根据需要修改 `.env` 中的配置（默认配置即可直接使用）

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 配置反向代理后访问管理面板

5. 使用默认账号登录：
   - 用户名：`admin`
   - 密码：`admin`

   > ⚠️ **安全提示**：首次登录后请立即修改默认密码

## 配置说明

### 基础配置

| 变量 | 说明 | 默认值 |
|-----|------|--------|
| `VERSION` | 镜像版本 | `latest` |
| `TZ` | 时区 | `Asia/Shanghai` |
| `OCTOPUS_SERVER_HOST` | 监听地址 | `0.0.0.0` |
| `OCTOPUS_SERVER_PORT` | 服务端口 | `8080` |

### 数据库配置

| 变量 | 说明 | 默认值 |
|-----|------|--------|
| `OCTOPUS_DATABASE_TYPE` | 数据库类型 | `sqlite` |
| `OCTOPUS_DATABASE_PATH` | 数据库路径 | `/app/data/data.db` |

**数据库类型说明：**

- **SQLite**（默认）：无需额外配置，数据存储在 Docker 卷中
- **MySQL**：需手动创建数据库，连接格式：`user:password@tcp(host:3306)/octopus`
- **PostgreSQL**：需手动创建数据库，连接格式：`postgresql://user:password@host:5432/octopus?sslmode=disable`

### 日志配置

| 变量 | 说明 | 可选值 |
|-----|------|--------|
| `OCTOPUS_LOG_LEVEL` | 日志级别 | `debug`、`info`、`warn`、`error` |

## 数据持久化

所有数据（数据库、配置文件、统计信息）存储在 Docker 命名卷 `octopus_octopus_data` 中：

```bash
# 查看卷位置
docker volume inspect octopus_octopus_data

# 备份数据
docker run --rm \
  -v octopus_octopus_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/octopus-backup-$(date +%Y%m%d).tar.gz -C /data .

# 恢复数据
docker run --rm \
  -v octopus_octopus_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/octopus-backup-YYYYMMDD.tar.gz -C /data
```

## 核心概念

### 渠道（Channel）

渠道是连接 LLM 提供商的基本配置单位。配置渠道时只需提供 Base URL，程序会自动根据渠道类型添加 API 路径：

| 渠道类型 | 自动添加路径 | Base URL 示例 |
|---------|-------------|--------------|
| OpenAI Chat | `/chat/completions` | `https://api.openai.com/v1` |
| OpenAI Responses | `/responses` | `https://api.openai.com/v1` |
| Anthropic | `/messages` | `https://api.anthropic.com/v1` |
| Gemini | `/models/:model:generateContent` | `https://generativelanguage.googleapis.com/v1beta` |

### 分组（Group）

分组将多个渠道聚合为一个统一的模型名称，实现负载均衡：

**负载均衡模式：**
- 🔄 **轮询**：按顺序循环使用每个渠道
- 🎲 **随机**：每次请求随机选择可用渠道
- 🛡️ **故障转移**：优先使用高优先级渠道，失败时切换
- ⚖️ **加权**：根据配置的权重分配请求

**示例：** 创建名为 `gpt-4o` 的分组，添加多个提供商的 GPT-4o 渠道，通过统一的 `model: gpt-4o` 访问所有渠道。

## 反向代理配置

服务默认在 Docker 网络内部暴露 8080 端口，需通过反向代理访问。

### Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name llm.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://octopus:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持（如需要）
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Caddy 配置示例

```caddyfile
llm.example.com {
    reverse_proxy octopus:8080
}
```

## 客户端集成

### OpenAI SDK（Python）

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://llm.example.com/v1",
    api_key="sk-octopus-YOUR_API_KEY"
)

response = client.chat.completions.create(
    model="octopus-gpt-4o",  # 使用分组名称
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)
```

### Claude Code

编辑 `~/.claude/settings.json`：

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://llm.example.com",
    "ANTHROPIC_AUTH_TOKEN": "sk-octopus-YOUR_API_KEY",
    "ANTHROPIC_MODEL": "octopus-sonnet-4-5"
  }
}
```

### Codex

编辑 `~/.codex/config.toml`：

```toml
model = "octopus-gpt-4o"
model_provider = "octopus"

[model_providers.octopus]
name = "octopus"
base_url = "https://llm.example.com/v1"
```

编辑 `~/.codex/auth.json`：

```json
{
  "OPENAI_API_KEY": "sk-octopus-YOUR_API_KEY"
}
```

## 系统要求

- **最小配置**：1 vCPU、512MB RAM、1GB 磁盘
- **生产推荐**：2+ vCPU、1-2GB RAM、SSD 10GB+

## 注意事项

1. **统计保存**：统计数据先存储在内存中，按配置的间隔批量写入数据库。停止容器时请使用 `docker compose down` 而非强制终止，以确保数据正确写入。

2. **价格管理**：系统自动从 models.dev 同步价格。如需覆盖默认价格，可在价格管理页面手动设置。

3. **首次登录**：必须修改默认密码以确保安全。

## 相关链接

- [GitHub 仓库](https://github.com/bestruirui/octopus)
- [官方文档](https://github.com/bestruirui/octopus#readme)
