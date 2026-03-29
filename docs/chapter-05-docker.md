# 第五章：封装——Docker 容器化与部署

> **核心哲学**：把环境装进"集装箱"，消灭"在我机器上能跑"的噩梦

---

## 5.1 容器的本质：集装箱哲学

Docker 不是虚拟机，而是"集装箱"。虚拟机模拟整台机器（CPU、内存、操作系统），启动慢、资源消耗大。容器只隔离进程，共享宿主机内核，启动毫秒级、资源消耗极小。

理解集装箱的比喻：你把货物（应用）装进标准箱子（容器），箱子可以在任何船上（服务器）运输。船不需要知道箱子里是什么，箱子不需要知道船是什么。标准化的接口让运输变得简单可靠。

### 容器 vs 虚拟机对比

| 维度 | 虚拟机 | 容器 |
|------|--------|------|
| 启动时间 | 分钟级 | 毫秒级 |
| 资源占用 | GB 级（完整 OS） | MB 级（共享内核） |
| 隔离级别 | 硬件级隔离 | 进程级隔离 |
| 移植性 | 需要相同虚拟化平台 | 任何 Linux 内核 |
| 适用场景 | 强隔离需求、多 OS | 微服务、快速部署 |

### 实战命令

```bash
# === 验证 Docker 安装 ===
docker --version                        # 查看版本
docker run hello-world                  # 测试运行

# === 基础操作 ===
docker ps                               # 运行中的容器
docker ps -a                            # 所有容器（含停止的）
docker images                           # 本地镜像列表
```

---

## 5.2 Docker 核心概念：镜像、容器、仓库

三个核心概念：镜像（Image）是蓝图，容器（Container）是实例，仓库（Registry）是分发中心。类比：镜像 = 类，容器 = 对象，仓库 = Git 服务器。

镜像是一层层叠加的，每层是只读的文件变更。容器在镜像顶层加一个可写层，所有运行时变更都写在这里。这种分层结构让镜像复用高效：多个镜像可共享底层。

### 镜像层级结构

```
┌─────────────────────┐
│   容器可写层         │  ← 运行时变更
├─────────────────────┤
│   应用层 (Layer 4)   │  ← 你的代码
├─────────────────────┤
│   依赖层 (Layer 3)   │  ← pip install
├─────────────────────┤
│   运行时层 (Layer 2) │  ← Python 环境
├─────────────────────┤
│   基础层 (Layer 1)   │  ← ubuntu:22.04
└─────────────────────┘
```

### 实战命令

```bash
# === 镜像操作 ===
docker pull python:3.12                 # 拉取镜像
docker images                           # 查看本地镜像
docker rmi python:3.12                  # 删除镜像
docker image prune                      # 清理未使用的镜像

# === 容器操作 ===
docker run -it python:3.12 bash         # 运行并进入容器
docker run -d --name web nginx          # 后台运行
docker stop web                         # 停止容器
docker rm web                           # 删除容器
docker container prune                  # 清理停止的容器

# === 查看详情 ===
docker inspect web                      # 容器详细信息
docker logs web                         # 容器日志
docker exec -it web bash                # 进入运行中的容器
```

---

## 5.3 Dockerfile：镜像的蓝图

Dockerfile 是构建镜像的脚本。每条指令创建一个层，从基础镜像逐步叠加到最终镜像。写得好的 Dockerfile 能让镜像体积小、构建快、缓存利用率高。

核心原则：把变化少的指令放前面（利用缓存），把变化多的放后面；合并 RUN 指令减少层数；清理临时文件避免残留。

### Dockerfile 指令对比

| 指令 | 作用 | 推荐用法 |
|------|------|----------|
| FROM | 基础镜像 | 选择最小镜像（python:3.12-slim） |
| WORKDIR | 工作目录 | 绝对路径，避免 cd |
| COPY | 复制文件 | 只复制必要文件 |
| RUN | 执行命令 | 合并多条，清理缓存 |
| ENV | 环境变量 | 配置参数 |
| EXPOSE | 暴露端口 | 声明端口（不实际开放） |
| CMD | 默认命令 | 容器启动时执行 |
| ENTRYPOINT | 入口命令 | 固定命令，CMD 作为参数 |

### 实战：标准 Python Dockerfile

```dockerfile
# === 基础镜像（选择 slim 版）===
FROM python:3.12-slim

# === 设置工作目录 ===
WORKDIR /app

# === 安装依赖（先复制依赖文件，利用缓存）===
COPY pyproject.toml uv.lock ./
RUN pip install uv && \
    uv sync --frozen --no-dev && \
    rm -rf ~/.cache

# === 复制代码（最后复制，变化最多）===
COPY src/ ./src/

# === 环境变量 ===
ENV PYTHONUNBUFFERED=1

# === 暴露端口 ===
EXPOSE 8000

# === 启动命令 ===
CMD ["uv", "run", "python", "src/main.py"]
```

### 实战命令

```bash
# === 构建镜像 ===
docker build -t myapp:v1 .              # 构建并命名
docker build -t myapp:v1 -f Dockerfile.prod .  # 指定 Dockerfile

# === 查看构建历史 ===
docker history myapp:v1                 # 查看各层大小

# === 运行镜像 ===
docker run -d -p 8000:8000 --name app myapp:v1
```

---

## 5.4 docker-compose：多容器编排

单容器用 `docker run`，多容器用 `docker-compose`。Compose 用 YAML 文件定义多个服务、网络、卷的配置，一条命令启动整个应用栈。

Compose 的核心价值：把复杂的启动命令变成声明式配置；服务间依赖自动管理；开发/生产环境配置一致。

### Compose 文件结构

```yaml
# docker-compose.yml
services:
  web:                                  # 服务名称
    build: .                            # 从当前目录构建
    ports:
      - "8000:8000"                     # 端口映射
    environment:
      - DATABASE_URL=postgres://db:5432
    depends_on:
      - db                              # 依赖 db 服务
    volumes:
      - ./src:/app/src                  # 开发时热重载

  db:                                   # 数据库服务
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data  # 数据持久化

volumes:
  pgdata:                               # 定义卷

networks:
  default:                              # 默认网络（自动创建）
    name: app-network
```

### Compose 常用指令

| 指令 | 作用 | 示例 |
|------|------|------|
| build | 构建镜像 | `build: .` 或 `build: ./backend` |
| image | 使用已有镜像 | `image: nginx:latest` |
| ports | 端口映射 | `"8000:80"`（宿主机:容器） |
| volumes | 挂载卷 | `./data:/app/data` |
| environment | 环境变量 | `KEY: value` |
| depends_on | 启动依赖 | `depends_on: [db]` |
| restart | 重启策略 | `always` / `on-failure` |

### 实战命令

```bash
# === 启动整个栈 ===
docker-compose up                       # 前台启动
docker-compose up -d                    # 后台启动
docker-compose up --build               # 重新构建后启动

# === 管理 ===
docker-compose down                     # 停止并删除
docker-compose down -v                  # 同时删除卷
docker-compose ps                       # 查看状态
docker-compose logs                     # 查看日志
docker-compose logs -f web              # 跟踪单个服务日志

# === 单服务操作 ===
docker-compose stop web                 # 停止单个服务
docker-compose restart web              # 重启单个服务
docker-compose exec web bash            # 进入服务容器
```

---

## 5.5 数据管理：Volume 与 Bind Mount

容器默认 ephemeral（短暂的）：删除容器后数据丢失。持久化数据用 Volume（Docker 管理）或 Bind Mount（宿主机路径）。

选择原则：生产数据用 Volume（Docker 管理，更安全）；开发代码用 Bind Mount（热重载）；配置文件用 Bind Mount（方便编辑）。

### 数据持久化方式对比

| 方式 | Docker 管理 | 适用场景 | 性能 |
|------|-------------|----------|------|
| Volume | ✅ 是 | 数据库、生产数据 | 高（Docker 优化） |
| Bind Mount | ❌ 否 | 开发代码、配置 | 中（依赖宿主机） |
| tmpfs | 内存 | 临时缓存、敏感数据 | 最高（不持久化） |

### 实战命令

```bash
# === Volume 操作 ===
docker volume create mydata             # 创建卷
docker volume ls                        # 列出卷
docker volume inspect mydata            # 查看详情
docker volume rm mydata                 # 删除卷
docker volume prune                     # 清理未使用卷

# === 使用 Volume ===
docker run -v mydata:/app/data myapp    # 挂载命名卷
docker run -v pgdata:/var/lib/postgresql/data postgres

# === Bind Mount ===
docker run -v ./src:/app/src myapp      # 挂载宿主机目录
docker run -v $(pwd)/config.yml:/app/config.yml myapp

# === tmpfs（临时内存盘）===
docker run --tmpfs /tmp:rw myapp        # 内存临时盘
```

---

## 5.6 网络：容器间通信

容器间通信通过网络。默认网络是 `bridge`，同一网络内的容器可以互相访问（用服务名作为 hostname）。自定义网络可以隔离不同应用的容器。

生产环境推荐：每个应用栈使用独立网络；数据库不暴露端口到宿主机；通过服务名访问，不使用 IP。

### 网络类型对比

| 类型 | 特点 | 适用场景 |
|------|------|----------|
| bridge | 默认，单主机隔离 | 单机多容器 |
| host | 共享宿主机网络 | 高性能、无隔离 |
| none | 无网络 | 安全隔离 |
| overlay | 跨主机通信 | Swarm / Kubernetes |

### 实战命令

```bash
# === 网络操作 ===
docker network create app-network       # 创建网络
docker network ls                       # 列出网络
docker network inspect app-network      # 查看详情
docker network rm app-network           # 删除网络

# === 连接网络 ===
docker run --network app-network --name web myapp
docker network connect app-network existing-container

# === 容器间通信 ===
# 同一网络内，容器可以用服务名互访：
# web 容器访问 db 容器：curl http://db:5432
```

### Compose 网络配置

```yaml
services:
  web:
    networks:
      - frontend
      - backend        # 连接多个网络

  db:
    networks:
      - backend        # 只在 backend 网络

networks:
  frontend:
  backend:
```

---

## 5.7 镜像优化：减小体积

镜像体积直接影响拉取速度、存储成本、部署效率。优化原则：选小基础镜像、合并层、清理缓存、多阶段构建。

一个典型 Python 应用：未优化镜像 ~1GB，优化后 ~100MB。多阶段构建是最强大的优化手段：构建阶段用完整镜像，运行阶段用精简镜像。

### 优化技巧对比

| 技巧 | 效果 | 操作 |
|------|------|------|
| 选择 slim 镜像 | 减少 50-70% | `python:3.12-slim` 替代 `python:3.12` |
| 合并 RUN 指令 | 减少层数 | `RUN apt update && apt install ... && apt clean` |
| 清理缓存 | 减少 10-30% | `rm -rf ~/.cache /var/lib/apt/lists/*` |
| 多阶段构建 | 减少 80%+ | 构建阶段和运行阶段分离 |
| .dockerignore | 避免复制无关文件 | 排除 `.git`、`__pycache__` |

### 实战：多阶段构建

```dockerfile
# === 阶段 1：构建 ===
FROM python:3.12 AS builder
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen
COPY src/ ./src/

# === 阶段 2：运行 ===
FROM python:3.12-slim
WORKDIR /app
COPY --from=builder /app/.venv /app/.venv
COPY --from=builder /app/src /app/src
ENV PATH="/app/.venv/bin:$PATH"
CMD ["python", "src/main.py"]
```

### .dockerignore 示例

```dockerignore
.git
.gitignore
__pycache__
*.pyc
.venv
node_modules
.env
tests/
docs/
*.md
```

### 实战命令

```bash
# === 查看镜像大小 ===
docker images                          # 查看所有镜像大小
docker history myapp:v1                # 查看各层大小

# === 对比优化前后 ===
docker build -t myapp:fat .            # 未优化版本
docker build -t myapp:slim -f Dockerfile.slim .  # 优化版本
docker images | grep myapp             # 对比大小
```

---

## 5.8 完整部署案例

场景：部署一个 FastAPI + PostgreSQL 的 Web 应用。

```yaml
# docker-compose.yml
services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8081:8000"
    environment:
      - DATABASE_URL=postgresql://app:secret@db:5432/appdb
      - MODEL_PROVIDER=deepseek
    depends_on:
      db:
        condition: service_healthy      # 等待数据库健康
    volumes:
      - ./src:/app/src                  # 开发热重载
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  pgdata:

networks:
  default:
    name: app-network
```

```dockerfile
# Dockerfile
FROM python:3.12-slim

WORKDIR /app

# 安装依赖
COPY pyproject.toml uv.lock ./
RUN pip install uv --quiet && \
    uv sync --frozen --no-dev && \
    rm -rf ~/.cache /root/.cache

# 复制代码
COPY service/ ./service/
COPY src/ ./src/

ENV PYTHONUNBUFFERED=1
EXPOSE 8000

CMD ["uv", "run", "python", "-m", "service.server"]
```

```bash
# === 部署流程 ===
# 1. 构建并启动
docker-compose up -d --build

# 2. 查看状态
docker-compose ps

# 3. 查看日志
docker-compose logs -f api

# 4. 健康检查
curl http://localhost:8081/health

# 5. 停止服务
docker-compose down

# 6. 清理数据（谨慎）
docker-compose down -v
```

---

## 5.9 开发 vs 生产配置

开发环境和生产环境需求不同：开发需要热重载、调试工具；生产需要稳定、安全、资源限制。最佳实践：使用不同的 compose 文件。

### 配置差异对比

| 配置项 | 开发环境 | 生产环境 |
|--------|----------|----------|
| 代码挂载 | Bind Mount（热重载） | COPY 到镜像 |
| 端口暴露 | 全部暴露 | 只暴露必要端口 |
| 环境变量 | .env 文件 | 环境变量注入 |
| 资源限制 | 无限制 | CPU/内存限制 |
| 日志 | 终端输出 | 日志驱动 |
| 重启策略 | 手动重启 | `restart: always` |

### 实战：多 compose 文件

```bash
# === 开发环境 ===
docker-compose -f docker-compose.yml up -d

# === 生产环境 ===
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

```yaml
# docker-compose.prod.yml（生产覆盖配置）
services:
  api:
    volumes: []                         # 移除开发挂载
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 5.10 关键认知

三个核心原则：

1. **容器不是虚拟机**：共享内核，进程隔离，启动毫秒级
2. **镜像分层是关键**：变化少的放前面，变化多的放后面
3. **生产数据用 Volume**：Bind Mount 用于开发，Volume 用于持久化

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 镜像太大 | 拉取慢、部署慢 | 多阶段构建 + slim 镜像 |
| 数据丢失 | 删除容器后数据消失 | 使用 Volume 持久化 |
| 网络不通 | 容器间无法通信 | 同一网络，用服务名访问 |
| 缓存失效 | 构建慢、每次重新安装 | 依赖文件先 COPY，代码后 COPY |
| 开发配置进生产 | 安全风险、资源浪费 | 分离 compose 文件 |
| 忘记清理 | 磁盘空间不足 | `docker system prune` |

### 实战命令

```bash
# === 清理 Docker 系统 ===
docker system prune                     # 清理未使用资源
docker system prune -a                  # 清理所有未使用镜像
docker system df                        # 查看磁盘使用

# === 常用诊断 ===
docker stats                            # 实时资源使用
docker inspect container                # 容器详情
docker network inspect network          # 网络详情
```

---

> **下一章**：掌握了容器化部署后，我们将进入 AI 原生开发时代，学习如何与 AI 协作编程。