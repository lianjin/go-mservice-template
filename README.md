# go-mservice-template

Go 微服务模板仓库，基于 `common` / `client` / `server` 三模块结构，支持一键从 GitHub Template 创建新服务并自动替换占位符。

## 项目结构

```
.
├── common/          # protobuf、gRPC 公共定义
├── client/          # gRPC 客户端 SDK
├── server/          # HTTP + gRPC 主服务
├── scripts/         # 初始化与 CI 辅助脚本
└── .github/workflows/
```

## 模板占位符

创建新仓库前，以下内容使用统一占位符，由 `init-from-template` workflow 自动替换：

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `__TEMPLATE_ORG__` | GitHub org / 用户名 | `lianjin` |
| `__TEMPLATE_REPO__` | 仓库名 | `order-mservice` |
| `__GO_VERSION__` | Go 版本 | `1.25.10` |
| `__SERVICE_SLUG__` | HTTP 路由前缀 | `order-ms` |
| `__DB_NAME__` | MySQL 数据库名 | `order_ms_db` |
| `__SONAR_PROJECT_KEY__` | Sonar 项目 Key | `lianjin_order-mservice` |
| `__PROTO_FILE__` | proto 文件名（domain） | `order`（来自 `order-mservice`） |
| `__PROTO_PACKAGE__` | proto Go 包名 | `orderpb` |
| `__GRPC_SERVICE__` | gRPC service 名称 | `OrderService` |

仓库名约定为 `<domain>-mservice`，初始化时从 repo 名去掉 `-mservice` 得到 domain，再推导 proto 相关命名。

模块路径格式：

```
github.com/__TEMPLATE_ORG__/__TEMPLATE_REPO__/{common|client|server}
```

## 创建新微服务

### 方式一：GitHub Template（推荐）

1. 点击 **Use this template** 创建新仓库
2. 首次 push 到 `main` 后，`Init from template` workflow 自动运行
3. 自动将占位符替换为：
   - org → 仓库 owner
   - repo → 仓库名
   - go_version → 默认 `1.25.10`
   - service_slug / db_name → 由仓库名推导
4. 完成后会提交 `chore: initialize microservice from template`

若 push 失败并提示 `without workflows permission`，需确认：
- `init-from-template.yml` 已声明 `permissions.workflows: write`（模板最新版已包含）
- 组织 **Settings → Actions → General → Workflow permissions** 为 **Read and write permissions**（不要选只读）

### 方式二：手动触发

在新仓库中打开 **Actions → Init from template → Run workflow**，可自定义：

- `org` / `repo`
- `go_version`
- `service_slug`
- `db_name`

### 方式三：从模板仓库远程触发

在模板仓库 `lianjin/go-mservice-template` 中运行 **Provision microservice** workflow，向目标仓库发送 `repository_dispatch` 事件。

需要在模板仓库配置 secret：

- `REPO_DISPATCH_TOKEN`：对目标仓库有 `contents` 写权限的 PAT

### 本地替换（可选）

```bash
chmod +x scripts/replace-template-vars.sh
./scripts/replace-template-vars.sh <org> <repo> <go_version> [service_slug] [db_name]
```

## CI/CD Workflows

参考 `campaign-center-api` 配置，包含：

| Workflow | 说明 |
|----------|------|
| `build.yml` | 全模块 build + test + golangci-lint |
| `codeql.yml` | CodeQL 静态分析 |
| `sonar.yml` | SonarQube（需配置 secrets） |
| `snyk.yml` | Snyk 依赖扫描（需配置 secrets） |
| `trivy.yml` | Trivy 文件系统扫描 |
| `deploy.yml` | Railway 手动部署 |
| `init-from-template.yml` | 占位符自动替换（新仓库初始化） |
| `provision-microservice.yml` | 从模板仓库远程触发初始化 |

## 本地开发

```bash
# 先完成占位符替换，或手动指定环境变量后启动
export MYSQL_PASSWORD=your_password

cd server && go run main.go
```

默认 HTTP 路由：

- `GET /__SERVICE_SLUG__/v1/ping`
- Swagger: `/__SERVICE_SLUG__/v1/swagger/index.html`

gRPC 默认端口：`5001`，HTTP 默认端口：`8080`。

## 依赖说明

`client` / `server` 通过 `replace` 指向本地 `../common`，便于单仓库开发。
