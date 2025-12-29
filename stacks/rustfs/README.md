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

## 访问密钥管理

RustFS 兼容 AWS S3 IAM 策略格式。在控制台创建访问密钥时，可通过策略限制密钥的访问范围。

### 策略格式

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:操作名称"],
      "Resource": ["arn:aws:s3:::存储桶名称/*"]
    }
  ]
}
```

**字段说明：**

| 字段 | 说明 |
|------|------|
| `Version` | 固定值 `2012-10-17` |
| `Effect` | `Allow`（允许）或 `Deny`（拒绝），Deny 优先级更高 |
| `Action` | S3 操作，支持通配符 `*` |
| `Resource` | 目标资源 ARN，格式 `arn:aws:s3:::bucket-name/prefix/*` |

### 常用策略示例

#### 单个存储桶完全访问

为密钥授予对 `my-bucket` 存储桶的完全控制权限：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:*"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

> 注意：需要同时指定存储桶本身（`my-bucket`）和桶内对象（`my-bucket/*`）两个资源。

#### 只读访问

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

#### 上传下载（无删除权限）

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

#### 动态策略（按用户名自动关联存储桶）

使用策略变量 `${aws:username}`，让访问密钥自动只能访问与其用户名同名的存储桶：

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:*"],
      "Resource": [
        "arn:aws:s3:::${aws:username}",
        "arn:aws:s3:::${aws:username}/*"
      ]
    }
  ]
}
```

**使用场景**：创建名为 `app-backup` 的存储桶，然后创建用户名也为 `app-backup` 的访问密钥并应用此策略。该密钥将自动只能访问 `app-backup` 存储桶。

> 这是推荐的做法：创建一个通用策略，然后通过用户名与存储桶名的对应关系实现权限隔离，无需为每个存储桶单独编写策略。

### 策略变量

| 变量 | 说明 |
|------|------|
| `${aws:username}` | 当前访问密钥所属的用户名 |
| `${aws:userid}` | 用户唯一 ID |

### 常用 Action 列表

| Action | 说明 |
|--------|------|
| `s3:*` | 所有操作 |
| `s3:GetObject` | 下载对象 |
| `s3:PutObject` | 上传对象 |
| `s3:DeleteObject` | 删除对象 |
| `s3:ListBucket` | 列出存储桶内容 |
| `s3:GetBucketLocation` | 获取存储桶位置 |
| `s3:ListAllMyBuckets` | 列出所有存储桶 |

## 相关链接

- [项目主页](https://github.com/rustfs/rustfs)
- [官方文档](https://docs.rustfs.com)
- [中文文档](https://docs.rustfs.com.cn)
- [Docker 部署文档](https://docs.rustfs.com.cn/installation/docker/)
