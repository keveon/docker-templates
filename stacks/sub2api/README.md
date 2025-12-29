# Sub2API

订阅转换 API 服务，支持多种代理协议转换和 AI 模型接口聚合。

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 编辑 `.env` 文件，设置必要的配置项（至少设置数据库密码）

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 配置反向代理后访问服务

## 配置说明

### 基础配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `TZ` | 时区 | `Asia/Shanghai` |
| `RUN_MODE` | 运行模式 (standard/simple) | `standard` |
| `SERVER_MODE` | 服务模式 (debug/release) | `release` |

### 数据库配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `DATABASE_HOST` | PostgreSQL 主机 | `postgres` |
| `DATABASE_PORT` | PostgreSQL 端口 | `5432` |
| `DATABASE_DBNAME` | 数据库名 | `sub2api` |
| `DATABASE_USER` | 数据库用户 | `sub2api` |
| `DATABASE_PASSWORD` | 数据库密码 | **必填** |

### Redis 配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `REDIS_HOST` | Redis 主机 | `redis` |
| `REDIS_PORT` | Redis 端口 | `6379` |
| `REDIS_PASSWORD` | Redis 密码 | 可选 |
| `REDIS_DB` | Redis 数据库 | `0` |

### 管理员配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `ADMIN_EMAIL` | 管理员邮箱 | `admin@example.com` |
| `ADMIN_PASSWORD` | 管理员密码 | 留空则自动生成 |

## 运行模式

- **standard**: 完整 SaaS 功能，包含计费/余额校验
- **simple**: 隐藏 SaaS 功能，跳过计费/余额校验（适合内部自用）

## 反向代理配置

服务默认只在 Docker 网络内部暴露 8080 端口，需通过反向代理访问。

### Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://sub2api:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 备份说明

数据存储在三个 Docker 卷中：

```bash
# 备份所有数据
docker run --rm \
    -v sub2api_sub2api_data:/sub2api \
    -v sub2api_postgres_data:/postgres \
    -v sub2api_redis_data:/redis \
    -v $(pwd):/backup alpine \
    tar czf /backup/sub2api-backup-$(date +%Y%m%d).tar.gz \
    -C / sub2api postgres redis

# 恢复数据
docker run --rm \
    -v sub2api_sub2api_data:/sub2api \
    -v sub2api_postgres_data:/postgres \
    -v sub2api_redis_data:/redis \
    -v $(pwd):/backup alpine \
    tar xzf /backup/sub2api-backup-YYYYMMDD.tar.gz -C /
```

## 系统要求

- **最小配置**: 1 vCPU, 1GB RAM, 5GB 磁盘
- **生产推荐**: 2+ vCPU, 2-4GB RAM, SSD 20GB+

## 相关链接

- [GitHub 仓库](https://github.com/weishaw/sub2api)
- [Docker Hub](https://hub.docker.com/r/weishaw/sub2api)
