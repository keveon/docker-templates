# Dozzle

轻量级实时 Docker 容器日志查看器，通过 Web 界面监控容器日志。

## 部署模式

| 模式 | 文件 | 说明 |
|-----|------|------|
| 主服务 | compose.yaml | 提供 Web UI，可连接远程 Agent |
| Agent | compose.agent.yaml | 仅作为被连接端，供远程 Dozzle 访问 |

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 根据需要修改 `.env` 中的配置

3. 启动服务：
   ```bash
   # 主服务模式
   docker compose up -d

   # Agent 模式
   docker compose -f compose.agent.yaml up -d
   ```

4. 通过反向代理访问服务
   - 主服务：端口 8080
   - Agent：端口 7007

## 配置说明

### 基础配置

| 变量 | 说明 | 默认值 |
|-----|------|--------|
| VERSION | 镜像版本 | latest |
| TZ | 时区 | Asia/Shanghai |

### 功能配置

| 变量 | 说明 | 默认值 |
|-----|------|--------|
| DOZZLE_NO_ANALYTICS | 禁用匿名分析 | true |
| DOZZLE_FILTER | 容器过滤器 | - |
| DOZZLE_HOSTNAME | 自定义主机名显示 | - |
| DOZZLE_LEVEL | 日志级别 | info |
| DOZZLE_ENABLE_ACTIONS | 启用容器操作 | false |

### Agent 配置（主服务模式）

| 变量 | 说明 | 示例 |
|-----|------|------|
| DOZZLE_REMOTE_AGENT | 连接远程 Agent | agent1:7007,agent2:7007 |

### 认证配置

| 变量 | 说明 | 可选值 |
|-----|------|--------|
| DOZZLE_AUTH_PROVIDER | 认证方式 | none, simple, forward-proxy |

## 多主机监控

1. 在远程主机部署 Agent：
   ```bash
   docker compose -f compose.agent.yaml up -d
   ```

2. 在主服务配置远程 Agent：
   ```env
   DOZZLE_REMOTE_AGENT=host1:7007,host2:7007
   ```

3. 启动主服务：
   ```bash
   docker compose up -d
   ```

## 反向代理配置

### Nginx（主服务）

```nginx
location / {
    proxy_pass http://dozzle:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

## 相关链接

- [官方网站](https://dozzle.dev)
- [GitHub](https://github.com/amir20/dozzle)
- [Agent 文档](https://dozzle.dev/guide/agent)
