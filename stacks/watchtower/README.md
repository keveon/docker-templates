# Watchtower

自动更新 Docker 容器镜像的工具，监控运行中的容器并在检测到新镜像时自动更新。

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 根据需要修改 `.env` 中的配置

3. 启动服务：
   ```bash
   docker compose up -d
   ```

## 配置说明

| 变量 | 默认值 | 说明 |
|------|--------|------|
| TZ | Asia/Shanghai | 时区 |
| WATCHTOWER_POLL_INTERVAL | 86400 | 检查更新间隔（秒） |
| WATCHTOWER_CLEANUP | false | 更新后删除旧镜像 |
| WATCHTOWER_LABEL_ENABLE | false | 只更新有标签的容器 |
| WATCHTOWER_INCLUDE_STOPPED | false | 包含已停止的容器 |
| WATCHTOWER_ROLLING_RESTART | false | 滚动更新 |

## 选择性更新

如果只想更新特定容器，设置 `WATCHTOWER_LABEL_ENABLE=true`，然后在目标容器上添加标签：

```yaml
services:
  your-service:
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
```

## 排除容器

在不想更新的容器上添加标签：

```yaml
services:
  your-service:
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
```

## 相关链接

- [项目主页](https://github.com/containrrr/watchtower)
- [官方文档](https://containrrr.dev/watchtower/)
