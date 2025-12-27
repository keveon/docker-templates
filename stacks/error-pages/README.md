# Error Pages

美观的 HTTP 错误页面服务，支持多种主题模板。可配合 Traefik 作为全局错误页面处理中间件。

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
| LOG_LEVEL | info | 日志级别：debug, info, warn, error |
| LOG_FORMAT | json | 日志格式：json, console |
| SHOW_DETAILS | true | 是否显示错误详情 |
| TEMPLATE_NAME | lost-in-space | 页面主题 |

## 可用主题

- `lost-in-space` - 太空主题（默认）
- `connection` - 连接断开主题
- `ghost` - 幽灵主题
- `shuffle` - 随机主题
- 更多主题见 [官方预览](https://tarampampam.github.io/error-pages/)

## Traefik 集成

本配置已包含 Traefik 标签，可作为全局错误页面中间件：

- 匹配所有域名（优先级最低）
- 处理 400-599 状态码
- 自动重定向到对应错误页面

在其他服务中启用错误页面中间件：
```yaml
labels:
  traefik.http.routers.your-service.middlewares: error-pages
```

## 相关链接

- [项目主页](https://github.com/tarampampam/error-pages)
- [可用主题预览](https://tarampampam.github.io/error-pages/)
- [Docker Hub](https://hub.docker.com/r/tarampampam/error-pages)
