# Immich

高性能自托管照片和视频备份解决方案，直接从手机自动备份，支持 AI 面部识别、智能搜索等功能。

## 功能特性

- 📸 **自动备份** - 手机照片和视频自动备份到自建服务器
- 🤖 **AI 识别** - 面部识别、场景分类、智能标签
- 🔍 **智能搜索** - 按人物、地点、时间、内容搜索
- 📱 **多端同步** - iOS、Android、Web 端无缝同步
- 🎨 **在线编辑** - 裁剪、滤镜、调色等在线编辑功能
- 🔄 **实时同步** - 即时上传和下载，支持 Live Photos
- 📊 **统计面板** - 存储统计、拍摄统计、地图视图
- 🌐 **地图视图** - 基于地理位置的照片浏览
- 👥 **共享相册** - 与家人朋友共享照片
- 🔒 **隐私保护** - 完全自托管，数据安全可控

## 快速开始

1. 复制环境变量文件：
   ```bash
   cp env.example .env
   ```

2. 编辑 `.env` 文件，设置数据库密码（必填）：
   ```bash
   DB_PASSWORD=your_secure_password
   ```

3. 启动服务：
   ```bash
   docker compose up -d
   ```

4. 配置反向代理后访问服务

## 配置说明

### 基础配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `IMMICH_VERSION` | 镜像版本 | `release` |
| `TZ` | 时区 | `Asia/Shanghai` |
| `RESTART` | 重启策略 | `unless-stopped` |

### 数据库配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `DB_PASSWORD` | 数据库密码 | **必填** |
| `DB_USERNAME` | 数据库用户名 | `postgres` |
| `DB_DATABASE_NAME` | 数据库名称 | `immich` |
| `DB_HOSTNAME` | 数据库主机 | `database` |
| `DB_PORT` | 数据库端口 | `5432` |

### 存储路径

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `UPLOAD_LOCATION` | 媒体文件存储路径 | `./upload` |
| `DB_DATA_LOCATION` | 数据库存储路径 | `./postgres` |

**⚠️ 重要提示**：
- 数据库存储**必须**使用本地 SSD，不能使用网络共享存储（NFS、SMB 等）
- 媒体文件存储建议预留足够空间（预估原库容量的 10-20% 额外空间）
- Windows 用户必须使用 WSL2，且文件系统需支持 Unix 权限（EXT4、ZFS、APFS 等）

### Redis 配置

| 变量 | 说明 | 默认值 |
|------|------|--------|
| `REDIS_HOSTNAME` | Redis 主机 | `redis` |
| `REDIS_PORT` | Redis 端口 | `6379` |

## 反向代理配置

服务默认只在 Docker 网络内部暴露 2283 端口，需通过反向代理访问。

### Nginx 配置示例

```nginx
server {
    listen 443 ssl http2;
    server_name photos.example.com;

    # SSL 证书配置
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # SSL 推荐配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # 文件上传大小限制（根据需要调整）
    client_max_body_size 500M;

    location / {
        proxy_pass http://immich-server:2283;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket 支持（实时同步）
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 上传超时设置
        proxy_read_timeout 600s;
        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;

        # 缓冲区设置
        proxy_buffering off;
        proxy_request_buffering off;
    }
}
```

### Nginx Proxy Manager 配置

1. 在 NPM 中添加 Proxy Host
2. **Forward Hostname/IP**: `immich-server`
3. **Forward Port**: `2283`
4. **Custom Location** 替换默认 location 配置（参考上方 Nginx 配置）
5. 启用 **WebSocket Support**
6. **Custom Nginx Configuration** 添加：
   ```nginx
   client_max_body_size 500M;
   proxy_read_timeout 600s;
   proxy_connect_timeout 600s;
   proxy_send_timeout 600s;
   proxy_buffering off;
   proxy_request_buffering off;
   ```

### Caddy 配置示例

```caddyfile
photos.example.com {
    reverse_proxy immich-server:2283

    # 文件上传大小限制
    request_body {
        max_size 500MB
    }
}
```

Caddy 会自动配置 HTTPS 和 WebSocket 支持。

## 移动端配置

1. 下载 Immich 移动应用：
   - [iOS App Store](https://apps.apple.com/app/id1613937347)
   - [Android Google Play](https://play.google.com/store/apps/details?id=app.iliyan.trimmich)

2. 打开应用，输入服务器地址：`https://photos.example.com`

3. 创建管理员账号

4. 开始备份照片和视频

5. 配置自动备份设置（仅在 WiFi 下备份、后台备份等）

## 数据管理

### 查看数据位置

```bash
# 查看媒体文件存储位置
ls -lh ./upload/

# 查看数据库存储位置
ls -lh ./postgres/

# 查看 Docker 卷位置
docker volume inspect immich_immich_model_cache
```

### 备份数据

```bash
# 备份媒体文件
tar czf immich-upload-$(date +%Y%m%d).tar.gz ./upload/

# 备份数据库
docker compose exec database pg_dump -U postgres immich > immich-db-$(date +%Y%m%d).sql

# 完整备份（包含所有数据）
docker run --rm \
  -v immich_immich_model_cache:/cache \
  -v $(pwd)/upload:/upload \
  -v $(pwd)/postgres:/postgres \
  -v $(pwd):/backup \
  alpine tar czf /backup/immich-full-$(date +%Y%m%d).tar.gz -C / upload postgres cache
```

### 恢复数据

```bash
# 恢复媒体文件
tar xzf immich-upload-YYYYMMDD.tar.gz -C ./

# 恢复数据库
docker compose exec -T database psql -U postgres immich < immich-db-YYYYMMDD.sql

# 恢复完整备份
docker run --rm \
  -v immich_immich_model_cache:/cache \
  -v $(pwd)/upload:/upload \
  -v $(pwd)/postgres:/postgres \
  -v $(pwd):/backup \
  alpine tar xzf /backup/immich-full-YYYYMMDD.tar.gz -C /
```

### 数据迁移

如果需要迁移到新的服务器：

1. 停止服务：`docker compose down`
2. 备份所有数据（参考上面的备份命令）
3. 在新服务器上部署 Immich
4. 恢复数据到新服务器
5. 启动服务：`docker compose up -d`

## 更新服务

```bash
# 查看当前版本
docker compose images

# 更新镜像
docker compose pull

# 重启服务
docker compose up -d

# 查看更新后的版本
docker compose images
```

## 系统要求

### 最低配置

- **CPU**: 2 核心
- **RAM**: 4GB
- **存储**: 2GB 可用空间（数据库必须使用本地 SSD）
- **网络**: 稳定的网络连接

### 生产推荐

- **CPU**: 4+ 核心
- **RAM**: 6GB+（数据库至少 2GB，机器学习服务 2GB+）
- **存储**: SSD 100GB+（根据照片数量估算）
- **网络**: 高带宽网络（用于快速上传下载）

### 操作系统

- **推荐**: Linux（Ubuntu 20.04+, Debian 11+, CentOS 8+）
- **支持**: macOS, Windows（WSL2）

**⚠️ 注意**：
- Windows 系统数据库不能使用 NTFS 或 exFAT32
- 数据库存储必须支持 Unix 权限（EXT4、ZFS、APFS 等）
- 不支持网络共享存储（NFS、SMB）作为数据库存储

## 端口说明

| 端口 | 用途 | 外部访问 |
|------|------|---------|
| 2283 | Web 界面和 API | 通过反向代理 |
| 3003 | 机器学习服务 API | 否（仅内部） |
| 5432 | PostgreSQL | 否（仅内部） |
| 6379 | Redis | 否（仅内部） |

## 硬件加速（可选）

Immich 支持硬件加速以提高转码和机器学习性能：

### 转码加速
- **NVIDIA NVENC**: 用于 GPU 加速视频转码
- **Intel QuickSync**: 用于 Intel 集成显卡加速
- **VAAPI**: 用于 Linux AMD/Intel GPU 加速
- **Rockchip MPP**: 用于 RK3588 等 ARM 设备

### 机器学习加速
- **CUDA**: NVIDIA GPU 加速
- **ROCm**: AMD GPU 加速
- **ARMNN**: ARM 设备加速
- **OpenVINO**: Intel CPU/GPU 加速

详见官方文档：[硬件加速配置](https://immich.app/docs/features/ml-hardware-acceleration)

## 常见问题

### Q: 数据库初始化失败？

A: 确保数据库存储路径使用本地 SSD，且文件系统支持 Unix 权限（EXT4、ZFS、APFS）。Windows 用户请使用 WSL2。

### Q: 照片上传速度慢？

A: 检查以下几点：
1. 网络带宽是否足够
2. 反向代理配置是否正确
3. 上传大小限制是否足够（client_max_body_size）
4. 服务器磁盘性能是否良好

### Q: AI 识别功能不工作？

A: 机器学习服务需要足够的内存（推荐 2GB+）。
1. 检查服务器内存是否充足
2. 查看 immich-machine-learning 容器日志：`docker compose logs immich-machine-learning`
3. 检查模型缓存卷是否正常挂载

### Q: 移动端无法连接服务器？

A: 确保：
1. 反向代理配置正确
2. SSL 证书有效且未过期
3. WebSocket 支持已启用
4. 服务器防火墙允许 443 端口
5. 使用 HTTPS 而非 HTTP 访问

### Q: 如何启用 HTTPS？

A: 推荐使用反向代理配置 SSL：
1. 使用 Let's Encrypt 获取免费证书
2. 配置 Nginx/Caddy 反向代理
3. 使用 Nginx Proxy Manager 自动管理证书

### Q: 可以使用外部 PostgreSQL 和 Redis 吗？

A: 可以。修改 `.env` 文件中的数据库和 Redis 配置：
```bash
# 外部数据库
DB_HOSTNAME=your-db-host
DB_PORT=5432

# 外部 Redis
REDIS_HOSTNAME=your-redis-host
REDIS_PORT=6379
```

然后在 `compose.yaml` 中移除 database 和 redis 服务。

### Q: 如何存储到对象存储（S3/MinIO）？

A: Immich 支持将原始媒体文件存储到对象存储。详见官方文档：[存储模板配置](https://immich.app/docs/administration/storage-template)

### Q: 如何配置 OAuth 登录？

A: Immich 支持多种 OAuth 提供商（Google、Facebook、Apple 等）。详见官方文档：[OAuth 配置](https://immich.app/docs/features/oauth)

### Q: 内存占用过高怎么办？

A: 可以采取以下措施：
1. 限制 Docker 容器内存使用
2. 使用更小的机器学习模型
3. 禁用部分 AI 功能
4. 将机器学习服务部署到独立服务器

### Q: 如何批量导入已有照片？

A: 有以下几种方式：
1. 使用 Web 界面上传（适合少量照片）
2. 使用 Immich CLI 工具上传（适合大量照片）
3. 直接复制到 upload 目录后重启服务

## 性能优化建议

### 数据库优化
- 使用 SSD 存储
- 调整 PostgreSQL 配置参数（shared_buffers、work_mem 等）
- 定期执行 `VACUUM ANALYZE`

### 机器学习优化
- 启用硬件加速（CUDA、ARMNN 等）
- 预加载常用模型
- 增加机器学习服务内存限制

### 存储优化
- 定期清理缩略图和缓存
- 使用对象存储存储原始文件
- 压缩视频文件

## 相关链接

- [官方网站](https://immich.app)
- [GitHub 仓库](https://github.com/immich-app/immich)
- [官方文档](https://immich.app/docs)
- [论坛社区](https://github.com/immich-app/immich/discussions)
- [更新日志](https://github.com/immich-app/immich/releases)
