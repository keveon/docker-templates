# RustFS

高性能 S3 兼容对象存储系统，使用 Rust 构建。对于 4KB 小文件，性能比 MinIO 快 2.3 倍。

> 注意：RustFS 目前处于 alpha 阶段，生产环境使用前请进行充分测试。

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 修改 `.env` 中的认证信息（生产环境必须修改默认密钥）：
   ```bash
   RUSTFS_ACCESS_KEY=your-access-key
   RUSTFS_SECRET_KEY=your-secret-key
   ```

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 访问控制台：`http://localhost:9001`

## 配置说明

### 基础配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `RUSTFS_VERSION` | `latest` | 镜像版本 |
| `TZ` | `Asia/Shanghai` | 时区 |
| `RUSTFS_OBS_LOGGER_LEVEL` | `info` | 日志级别：trace, debug, info, warn, error |

### 服务配置

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `RUSTFS_DATA_PATH` | `data` | 数据存储路径，默认命名卷，可设置绝对路径 |
| `RUSTFS_ADDRESS` | `:9000` | S3 API 监听地址 |
| `RUSTFS_CONSOLE_ADDRESS` | `:9001` | 控制台监听地址 |
| `RUSTFS_CONSOLE_ENABLE` | `true` | 启用 Web 控制台 |

### 认证配置（必填）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `RUSTFS_ACCESS_KEY` | `rustfsadmin` | 访问密钥 |
| `RUSTFS_SECRET_KEY` | `rustfsadmin` | 私有密钥 |

### 域名配置（可选）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `RUSTFS_SERVER_DOMAINS` | - | 服务域名，用于虚拟主机样式访问 |
| `RUSTFS_CORS_ALLOWED_ORIGINS` | - | CORS 允许的来源域名 |

## 端口说明

| 端口 | 用途 |
|------|------|
| 9000 | S3 API 端点 |
| 9001 | Web 控制台 |

## 使用示例

使用 MinIO Client (mc) 连接：

```bash
# 安装 mc
brew install minio/stable/mc

# 配置别名
mc alias set rustfs http://localhost:9000 your-access-key your-secret-key

# 创建 bucket
mc mb rustfs/my-bucket

# 上传文件
mc cp file.txt rustfs/my-bucket/
```

## 相关链接

- [项目主页](https://github.com/rustfs/rustfs)
- [官方文档](https://docs.rustfs.com)
- [中文文档](https://docs.rustfs.com.cn)
- [Docker 部署文档](https://docs.rustfs.com.cn/installation/docker/)
