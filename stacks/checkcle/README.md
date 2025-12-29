# CheckCle

自托管的开源监控平台，提供全栈系统、应用和基础设施的实时监控。

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 根据需要编辑 `.env` 文件配置时区

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 配置反向代理后访问服务，使用默认凭证登录

## 配置说明

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `CHECKCLE_VERSION` | 镜像版本 | `latest` |
| `TZ` | 时区 | `Asia/Shanghai` |

## 默认凭证

首次登录使用以下凭证：

- **用户名**: `admin@example.com`
- **密码**: `Admin123456`

> **重要**: 首次登录后请立即修改默认密码！

## 功能特性

- HTTP/DNS/Ping/TCP/API 服务监控
- SSL 证书和域名到期监控
- 服务器基础设施监控（Linux/Windows）
- 事件追踪和维护计划安排
- 多渠道通知（邮件、Telegram、Discord、Slack）
- 公开状态页面
- Docker 容器监控

## 反向代理配置

服务默认只在 Docker 网络内部暴露 8090 端口，需通过反向代理访问。

### Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name monitor.example.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://checkcle:8090;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 备份说明

数据存储在 Docker 卷 `data` 中，备份方法：

```bash
# 备份
docker run --rm -v checkcle_data:/data -v $(pwd):/backup alpine \
    tar czf /backup/checkcle-backup-$(date +%Y%m%d).tar.gz -C /data .

# 恢复
docker run --rm -v checkcle_data:/data -v $(pwd):/backup alpine \
    tar xzf /backup/checkcle-backup-YYYYMMDD.tar.gz -C /data
```

## 系统要求

- **最小配置**: 1 vCPU, 500MB RAM, 2GB 磁盘
- **生产推荐**: 2+ vCPU, 2-4GB RAM, SSD 10GB+

## 相关链接

- [官方网站](https://checkcle.io)
- [官方文档](https://docs.checkcle.io)
- [GitHub 仓库](https://github.com/operacle/checkcle)
- [演示环境](https://demo.checkcle.io)
