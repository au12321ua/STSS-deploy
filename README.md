# STSS Deploy — 构建编排

STSS 平台的 Docker Compose 部署仓库。编排网关、认证、业务服务，通过
GHCR 镜像一键拉起全栈。

---

## 子系统仓库

| 子系统 | 仓库地址 | 说明 |
|---|---|---|
| **STSS-deploy** | [au12321ua/STSS-deploy](https://github.com/au12321ua/STSS-deploy) | 部署编排（本仓库） |
| **STSS-gateway** | [au12321ua/STSS-gateway](https://github.com/au12321ua/STSS-gateway) | Nginx 网关，Bearer 令牌校验、路由代理、限流、CORS |
| **group1-base** | [gau12321ua/group1-base](https://github.com/au12321ua/group1-base) | Auth Service + Info Service + Seed |
| **zjuse-schedule** | [uppi7/zjuse-schedule](https://github.com/uppi7/zjuse-schedule) | 排课系统 |
| **Smart-Class-Selection-backend** | [hhhjyz/Smart-Class-Selection-backend](https://github.com/hhhjyz/Smart-Class-Selection-backend) | 选课系统 |
| **STSS-Online-Testing** | [cpmores/STSS-Online-Testing.git](https://github.com/cpmores/STSS-Online-Testing.git) | 在线测试系统 |
| **STSS-forum-backend** | [zheng-cao04/STSS-forum-backend](https://github.com/zheng-cao04/STSS-forum-backend)| 论坛服务 |
| **SE-Group6** | [Sicrete/SE-Group6](https://github.com/Sicrete/SE-Group6) | 成绩服务 |
| **STSS-frontend** | [Aparecium77/SSTS-frontend](https://github.com/Aparecium77/SSTS-frontend) | SPA 前端（Vite + 端口 5173） |

> 各服务的 GHCR 镜像地址及本地构建覆盖详见 [docker-compose.yml](docker-compose.yml) 和 [docker-compose.override.yml.example](docker-compose.override.yml.example)。

---

## 系统架构

```
浏览器 (SPA) ──→ :8000 Nginx Gateway ──→ :8001 Auth Service（认证）
                                      ──→ :8002 Info Service（业务）
                                      ──→ :8003 Schedule Service（排课）
                                      ──→ :8004 Course Selection（选课）
                                      ──→ :8005 Forum Service（论坛）
                                      ──→ :8006 Online Test（在线测试）
                                      ──→ :8007 Grade Service（成绩）
```

- **网关**：Bearer 令牌校验、路由代理、限流、CORS、健康检查聚合
- **前端只需与网关通信**，永远不直接调用后端服务

---

## 快速开始

```bash
# 1. 克隆仓库
git clone https://github.com/au12321ua/STSS-deploy.git && cd STSS-deploy

# 2. 创建 .env（从模板复制后修改密钥）
cp .env.example .env

# 3. 一键启动（从 GHCR 拉取全部镜像）
docker compose up -d

# 4. 首次使用时初始化数据（角色、权限、管理员用户、Schedule 教室）
docker compose --profile seed up seed schedule_seed

# 5. 验证
curl http://localhost:8000/api/v1/health
```

### 本地构建后端镜像

如果后端源码在 sibling 目录：

```bash
cp docker-compose.override.yml.example docker-compose.override.yml
docker compose --profile local-build build gateway auth_service info_service seed schedule_service schedule_worker schedule_seed course-selection-api course-selection-worker forum_service online_test_logger online_test_service online_test_proctor_controller online_test_proctor_image grade_service
docker compose up -d
docker compose --profile seed up seed schedule_seed
```

### 启动占位服务

当部分业务组后端尚未就绪，用占位容器保持网络可解析：

```bash
docker compose --profile backend-placeholders up -d
```

---

## 环境变量

| 变量 | 默认值 | 说明 |
|---|---|---|
| `GATEWAY_PORT` | `8000` | 网关监听端口 |
| `GATEWAY_IMAGE` | `ghcr.io/au12321ua/stss-gateway:c3585b4` | Gateway 镜像，包含 `/api/v1/classrooms` 路由 |
| `GATEWAY_CLIENT_ID` | `gateway` | 网关服务账号 |
| `GATEWAY_CLIENT_SECRET` | `change-me-gateway-secret` | 网关服务账号密钥 |
| `CORS_ORIGINS` | `http://localhost:5173,…` | 允许的 CORS 域名 |
| `RATE_LIMIT_LOGIN` | `10r/m` | 登录限流速率 |
| `AUTH_SERVICE_URL` | `auth_service:8001` | Auth 上游地址 |
| `INFO_SERVICE_URL` | `info_service:8002` | Info 上游地址 |
| `SCHEDULE_SERVICE_URL` | `schedule_service:8003` | 排课上游地址 |
| `COURSE_SELECTION_SERVICE_URL` | `course-selection-api:8004` | 选课上游地址 |
| `FORUM_SERVICE_URL` | `forum_service:8005` | 论坛上游地址 |
| `ONLINE_TEST_SERVICE_URL` | `online_test_service:8006` | 测试上游地址 |
| `GRADE_SERVICE_URL` | `grade_service:8007` | 成绩上游地址 |
| `TOKEN_SECRET_KEY` | — | JWT 签名密钥，auth 和 info 必须一致 |
| `INFO_SERVICE_CLIENT_SECRET` | — | Info 调 Auth 的凭据 |
| `SCHEDULE_SERVICE_CLIENT_ID` | `schedule_service` | Schedule 服务账号 ID |
| `SCHEDULE_SERVICE_CLIENT_SECRET` | — | Schedule 调 Auth 的凭据 |
| `COURSE_SELECTION_SERVICE_CLIENT_ID` | `course_selection_service` | Course Selection 服务账号 ID |
| `COURSE_SELECTION_SERVICE_CLIENT_SECRET` | — | Course Selection 调 Auth 后访问 Info/Schedule 的凭据 |
| `SCHEDULE_IMAGE` | `ghcr.io/uppi7/zjuse-schedule:latest` | Schedule API/Worker/seed 镜像 |
| `SCHEDULE_MYSQL_PASSWORD` | `rootpassword` | Schedule MySQL root 密码 |
| `SCHEDULE_MYSQL_DB` | `course_arrange` | Schedule 数据库名 |
| `COURSE_SELECTION_IMAGE` | `ghcr.io/hhhjyz/stss-course-selection:95b523e` | Course Selection API/Worker 镜像 |
| `COURSE_SELECTION_PG_PASSWORD` | `cs_pwd` | Course Selection PostgreSQL 密码 |
| `COURSE_SELECTION_PG_DB` | `course_selection` | Course Selection 数据库名 |
| `COURSE_SELECTION_RABBITMQ_USER` | `stss` | RabbitMQ 用户名 |
| `COURSE_SELECTION_RABBITMQ_PASSWORD` | `stss` | RabbitMQ 密码 |
| `FORUM_IMAGE` | `ghcr.io/au12321ua/stss-forum:latest` | Forum Service 镜像 |
| `FORUM_SKIP_EXTERNAL_CHECKS` | `true` | Forum 是否跳过外部选课/成绩服务强校验 |
| `FORUM_INTERNAL_TOKEN` | `dev-internal-token` | Forum 内部服务调用 token |
| `ENV` | `development` | 运行环境 |
| `LOG_LEVEL` | `DEBUG` | 日志级别 |
| `GRADE_DEV_MOCK_EXTERNAL` | `true` | Grade 模拟外部 API（开发用，避免依赖所有服务） |
| `AUTH_SERVICE_IMAGE` | `ghcr.io/au12321ua/stss-auth:latest` | Auth Service 镜像 |
| `INFO_SERVICE_IMAGE` | `ghcr.io/au12321ua/stss-info:latest` | Info Service 镜像 |
| `SEED_IMAGE` | `ghcr.io/au12321ua/stss-seed:latest` | Seed 一次性数据初始化镜像 |
| `FRONTEND_PATH` | `./frontend` | SPA 前端源码目录（dev profile） |
| `INTERNAL_TOKEN` | `dev-internal-token` | Forum 内部令牌（override 用，与 FORUM_INTERNAL_TOKEN 值相同） |
| `SCHEDULE_API_PORT` | `8003` | Schedule API 宿主机调试端口（override only） |
| `SCHEDULE_MYSQL_PORT` | `3309` | Schedule MySQL 宿主机调试端口（override only） |
| `SCHEDULE_REDIS_PORT` | `6382` | Schedule Redis 宿主机调试端口（override only） |

### 凭据一致性

```
GATEWAY_CLIENT_SECRET ────────────► auth_service: SERVICE_CLIENT_GATEWAY_SECRET
INFO_SERVICE_CLIENT_SECRET ───┬──► auth_service: SERVICE_CLIENT_INFO_SERVICE_SECRET
                              └──► info_service:  AUTH_SERVICE_CLIENT_SECRET
SCHEDULE_SERVICE_CLIENT_SECRET ─► auth_service: SERVICE_CLIENT_SCHEDULE_SERVICE_SECRET
COURSE_SELECTION_SERVICE_CLIENT_SECRET ─► auth_service: SERVICE_CLIENT_COURSE_SELECTION_SERVICE_SECRET
TOKEN_SECRET_KEY ───────┬────────► auth_service: TOKEN_SECRET_KEY
                        └────────► info_service:  TOKEN_SECRET_KEY
```

`.env` 中设置一次即可，compose 文件已通过变量引用保证一致性。

### Service Token 权限

最新版 `group1-base` 中，service token 的权限来自 Auth 数据库里的 `SERVICE` 角色，不再由
`SERVICE_CLIENT_*_SCOPE` 环境变量授权。Course Selection 需要该角色至少具备：

```
data-provision:read course:read offering:read schedule:read classroom:read
```

首次初始化时请运行 `docker compose --profile seed up seed schedule_seed`。如果本地已经有旧的
`auth_data` 卷，旧 seed 可能没有写入 `SERVICE` 角色或上述权限；此时需要重置/重新 seed
Auth 数据库，或在 Auth DB 中手动补齐 `SERVICE` 角色权限。单独添加
`SERVICE_CLIENT_COURSE_SELECTION_SERVICE_SCOPE` 不会解决权限问题。

---

## Profiles

| Profile | 说明 |
|---|---|
| （默认） | 启动 gateway + auth_service + info_service + schedule_service + schedule_worker + schedule_mysql + schedule_redis + course-selection-api + course-selection-worker + course-selection-pg + course-selection-redis + rabbitmq + forum_service |
| `seed` | 一次性初始化角色、权限、管理员用户、Schedule 教室数据 |
| `dev` | 附加 SPA 前端开发服务器（Vite，端口 5173） |
| `backend-placeholders` | 启动未接入业务的占位容器，让服务名可被网关 DNS 解析 |

---

## 如何接入新服务

### 概览

网关通过 **env var → upstream → location block** 三级路由。接入一个新业务服务需要在 **两个仓库** 各完成对应工作：

| 步骤 | 在哪里改 | 改什么 |
|---|---|---|
| 1. 服务实现 | 你自己的后端仓库 | 实现服务，满足基础要求 |
| 2. 网关路由 | `STSS-gateway` | 添加 env var + upstream + location |
| 3. 部署装配 | `STSS-deploy`（本仓库） | 添加 service 定义 + 环境变量 |

### 步骤 1：确保服务满足基础要求

→ 见下一节「服务基础要求」

### 步骤 2：在网关仓库添加路由

向 `STSS-gateway` 提交 PR，修改以下文件：

**a) `nginx/nginx.conf.template`** — 添加 env var 注释、upstream、resolver 变量、location block

以 `example_service`（端口 8008）为例：

```nginx
# 在文件头部 env var 注释区添加：
#   EXAMPLE_SERVICE_URL     Example upstream          (default: example_service:8008)

# 在 upstream 区添加：
upstream example_upstream {
    server ${EXAMPLE_SERVICE_URL};
    keepalive 32;
}

# 在 server 块的 resolver 区添加变量：
set $example_service_url "${EXAMPLE_SERVICE_URL}";

# 在 API location 区添加路由：
location ~ ^/api/v1/example(?:/|$) {
    auth_request /internal/auth/verify;
    proxy_pass   http://$example_service_url;
}
```

如果是 SSE 端点，需要额外关闭缓冲：

```nginx
location ~ ^/api/v1/example(?:/|$) {
    auth_request       /internal/auth/verify;
    proxy_pass         http://$example_service_url;
    proxy_buffering    off;
    proxy_read_timeout 3600s;
}
```

**b) `docker/entrypoint.sh`** — 添加默认值和 envsubst

```bash
export EXAMPLE_SERVICE_URL="${EXAMPLE_SERVICE_URL:-example_service:8008}"

# envsubst 列表里加 ${EXAMPLE_SERVICE_URL}
```

**c) 提交 PR，等待网关新镜像发布到 GHCR**

### 步骤 3：在本仓库装配

**a) `.env.example`** — 添加上游地址

```bash
EXAMPLE_SERVICE_URL=example_service:8008
```

**b) `docker-compose.yml`** — 添加服务定义

真实服务：

```yaml
services:
  example_service:
    image: ghcr.io/au12321ua/stss-example:latest
    container_name: stss-example
    expose:
      - "8008"
    environment:
      ENV:              "${ENV:-development}"
      TOKEN_SECRET_KEY: "${TOKEN_SECRET_KEY}"
      # … 服务自己的其他变量
    volumes:
      - example_data:/app/data
    restart: unless-stopped
    networks:
      - stss-net
    healthcheck:
      test:     ["CMD", "curl", "-f", "http://localhost:8008/api/v1/health"]
      interval: 15s
      timeout:  3s
      retries:  5
      start_period: 10s
```

占位容器（服务未就绪时先用）：

```yaml
  example_service:
    image: alpine:3.20
    container_name: stss-example-placeholder
    profiles: ["backend-placeholders"]
    command: ["sh", "-c", "echo 'example_service placeholder'; sleep infinity"]
    expose:
      - "8008"
    networks:
      - stss-net
```

**c) `gateway` 服务的 `environment` 段** — 传递新变量

```yaml
gateway:
  environment:
    # … 已有变量 …
    EXAMPLE_SERVICE_URL: "${EXAMPLE_SERVICE_URL:-example_service:8008}"
```

**d) 更新顶层 `volumes`**（如果需要持久化）

```yaml
volumes:
  # … 已有卷 …
  example_data:
```

---

## 服务基础要求

每个接入 STSS 网关的业务服务 **必须** 满足以下最低要求。

### 1. 健康检查端点

```
GET /api/v1/health
→ 200 OK
```

- 网关的聚合健康检查 `/api/v1/health` 会探测所有上游的 health endpoint
- 不要求鉴权（网关内部子请求不带用户 token）

### 2. 统一响应格式

所有 API 返回遵循：

```json
// 成功
{ "code": 0, "message": "success", "data": { ... } }

// 分页列表
{ "code": 0, "message": "success", "data": {
    "items": [ ... ],
    "pagination": { "total": 100, "page": 1, "page_size": 20 }
} }

// 错误
{ "code": 1001, "message": "error description", "errors": [{ "detail": "..." }] }
```

- `code === 0` → 成功
- `code !== 0` → 失败

### 3. 分页约定

列表类 GET 端点统一支持：

| 参数 | 类型 | 默认 | 范围 |
|---|---|---|---|
| `page` | int | 1 | ≥1 |
| `page_size` | int | 20 | 1–100 |
| `sort_by` | string | 因端点而异 | — |
| `sort_order` | string | asc | asc / desc |

### 4. 用户身份读取

网关完成鉴权后，将用户信息注入以下请求头。**服务不自行校验 Token**，只读请求头：

| 请求头 | 含义 |
|---|---|
| `X-User-Id` | 用户 ID |
| `X-User-Role` | 用户角色 |
| `X-User-Permissions` | 逗号分隔的权限列表 |
| `X-Request-ID` | 请求追踪 ID |

### 5. 鉴权方式

- **不自己校验 `Authorization: Bearer <token>`**
- 网关已校验完毕，未通过校验的请求不会到达服务
- 服务端根据 `X-User-Role` 和 `X-User-Permissions` 做权限控制即可
- 需要服务间调用时，使用 service client 凭据调用 `/auth/sys/login` 获取 service token

### 6. API 路径前缀

- 统一使用 `/api/v1/<module>/*` 格式，如 `/api/v1/example/items`
- 不要和其他模块的路径重叠

### 7. 不要设置 CORS 头

- CORS 由网关统一处理
- 服务端不要设置 `Access-Control-*` 响应头，避免冲突

### 8. Docker 镜像要求

- 基础镜像建议 `python:3.12-slim` 或 `alpine`（取决于技术栈）
- 容器监听 `0.0.0.0:<port>`
- 必须提供 healthcheck（推荐 HTTP `curl` 探测）
- 发布到 GHCR：`ghcr.io/au12321ua/stss-<name>:latest`
- 端口只 `expose`，不 `ports` 发布到宿主机（仅网关对外暴露）

### 9. 启动顺序

- 如果服务依赖 Auth Service（需要校验 token 或调用 `/auth/sys/login`），在 `docker-compose.yml` 中设置 `depends_on: auth_service` 并等待 `condition: service_healthy`

---

## 目录结构

```
STSS-deploy/
├── docker-compose.yml                  # 主 Compose 文件
├── docker-compose.override.yml.example # 本地构建覆盖
├── .env.example                        # 环境变量模板
├── .gitignore
├── gateway-backend-requirements.md     # 前后端联合需求文档
└── README.md
```

---

## 与 STSS-gateway 的关系

| 关注点 | STSS-gateway | STSS-deploy（本仓库） |
|---|---|---|
| Nginx 配置 & NJS 脚本 | ✅ | — |
| Dockerfile & 网关镜像构建 | ✅ | — |
| 网关 CI/CD（publish.yml） | ✅ | — |
| 服务编排（docker-compose） | — | ✅ |
| 环境变量装配 | — | ✅ |
| 各业务组服务定义 | — | ✅ |
| 前后端接口文档 | — | ✅ |
