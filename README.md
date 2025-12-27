# Docker Templates

常用服务的 Docker Compose 模板集合，开箱即用。

## 使用方法

1. 克隆仓库：
   ```bash
   git clone https://github.com/keveon/docker-templates.git
   cd docker-templates
   ```

2. 进入目标服务目录：
   ```bash
   cd stacks/<service-name>
   ```

3. 复制并配置环境变量：
   ```bash
   cp env.example .env
   # 编辑 .env 文件
   ```

4. 启动服务：
   ```bash
   docker compose up -d
   ```

## 可用模板

| 服务 | 说明 | 分类 |
|------|------|------|
| [error-pages](stacks/error-pages/) | 美观的 HTTP 错误页面服务 | 基础设施 |

## 目录结构

```
stacks/
└── <service>/
    ├── compose.yaml      # Docker Compose 配置
    ├── env.example       # 环境变量示例
    ├── README.md         # 使用说明
    └── metadata.yaml     # 元数据
```

## 贡献指南

欢迎提交 Pull Request 添加新模板，请确保：

1. 目录名使用小写字母和短横线
2. 包含完整的 `compose.yaml`、`env.example`、`README.md`、`metadata.yaml`
3. 环境变量带有清晰的注释说明
4. README 包含快速开始和配置说明

## 许可证

[MIT](LICENSE)
