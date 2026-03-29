# 第八章：实战——不同开发环境的差异与注意事项

> **核心哲学**：理解不同环境的差异，避免"本地能跑，服务器上跑不了"

---

## 8.1 三种开发环境对比

开发者通常会在三种环境中工作：个人电脑（本地）、云服务器（如华为云）、实验室/公司服务器。这三种环境差异很大，理解差异才能避免踩坑。

| 维度 | 个人电脑（本地） | 云服务器 | 实验室/共享服务器 |
|------|-----------------|----------|------------------|
| 所有权 | 完全自有 | 租赁（按需付费） | 共享（多人使用） |
| 权限 | root/管理员 | root 或受限 | 通常受限（普通用户） |
| 环境 | 自由配置 | 可重装系统 | 不能动系统环境 |
| 资源 | 有限（个人电脑） | 可扩展（付费） | 共享（需协调） |
| 网络 | 家庭/公司网络 | 公网IP、带宽稳定 | 内网、可能有防火墙 |
| 持久性 | 长期使用 | 按需启停 | 长期但可能被清理 |
| 成本 | 已购买（固定） | 按量计费 | 免费（但资源竞争） |

### 环境选择建议

| 场景 | 推荐环境 | 原因 |
|------|----------|------|
| 日常开发、学习 | 本地 | 方便、无成本、调试容易 |
| 需要 GPU 训练模型 | 云服务器（GPU 实例） | 本地 GPU 不足，云端可按需租用 |
| 长期运行服务 | 云服务器 | 稳定、公网可访问 |
| 科研计算（预算有限） | 实验室服务器 | 免费、可能有 GPU |
| 团队协作项目 | 云服务器 | 统一环境、权限可控 |
| 快速实验、原型验证 | 本地 | 启动快、迭代快 |

---

## 8.2 本地开发：自由但有限制

### 优势

- **完全控制**：想装什么装什么，想改什么改什么
- **调试方便**：IDE 断点、热重载、图形工具都可用
- **无成本**：不需要额外付费
- **隐私安全**：代码和数据在本地，不经过网络

### 限制

- **资源有限**：CPU、内存、GPU 都有上限
- **环境差异**：与生产环境可能不一致
- **网络限制**：家庭网络可能无公网 IP，外网访问困难
- **不可持久**：电脑关机后服务停止

### 本地开发最佳实践

```bash
# === 本地开发环境设置 ===

# 1. 使用虚拟环境隔离项目
cd ~/projects/myproject
uv sync

# 2. 用 .env 管理配置（不提交敏感信息）
cp .env.example .env
# 编辑 .env 填入真实配置

# 3. 本地开发用 Docker 跑依赖服务
docker run -d --name local-postgres \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:16-alpine

docker run -d --name local-redis \
  -p 6379:6379 \
  redis:7-alpine

# 4. 启动开发服务
uv run python run_server.py

# 5. 开发完成后清理
docker stop local-postgres local-redis
docker rm local-postgres local-redis
```

### 本地开发常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 环境不一致 | 本地能跑，部署失败 | 用 Docker 统一环境 |
| 依赖污染 | 多项目冲突 | 每个项目用独立 .venv |
| 端口冲突 | 服务启动失败 | 检查端口占用，修改配置 |
| 资源不足 | 训练慢、卡死 | 减少模型大小，或用云服务器 |
| 配置泄露 | 敏感信息提交到 Git | .gitignore 忽略 .env |

---

## 8.3 云服务器开发：稳定但需注意成本

### 优势

- **公网访问**：有公网 IP，服务可对外暴露
- **资源可扩展**：CPU、内存、GPU 可按需升级
- **环境可控**：可重装系统、自定义配置
- **稳定性高**：数据中心网络、电力稳定

### 常见云服务商对比

| 服务商 | 特点 | 适用场景 |
|--------|------|----------|
| 阿里云 | 国内最大，生态完善 | 国内业务首选 |
| 腾讯云 | 价格有竞争力 | 个人项目、创业公司 |
| 华为云 | 政企客户多，安全合规 | 企业级应用 |
| AWS | 全球最大，功能最全 | 海外业务 |
| AutoDL 等 GPU 云 | 便宜 GPU 租赁 | 深度学习训练 |

### 云服务器开发流程

```bash
# === 云服务器开发流程 ===

# 1. 连接服务器（SSH）
ssh user@your-server-ip

# 2. 首次登录：更新系统
sudo apt update && sudo apt upgrade -y

# 3. 安装开发工具
sudo apt install -y git curl python3 python3-pip python3-venv

# 4. 克隆项目
git clone https://github.com/user/project.git
cd project

# 5. 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 6. 配置环境变量
cp .env.example .env
nano .env  # 编辑配置

# 7. 启动服务（开发模式）
python run_server.py

# 8. 生产部署（用 systemd 或 Docker）
# 见后续章节
```

### 云服务器注意事项

**成本控制**：

| 项目 | 成本因素 | 节省技巧 |
|------|----------|----------|
| 计算资源 | CPU/内存配置 | 按需选择，非 24h 服务可停机 |
| 存储 | 磁盘大小、类型 | 定期清理无用数据，用对象存储存静态文件 |
| 带宽 | 公网带宽 | 小项目用小带宽，大文件用 CDN |
| GPU | GPU 型号、时长 | 训练完立即释放，用竞价实例 |

```bash
# === 成本控制技巧 ===

# 1. 查看云服务器费用明细
# 在云控制台查看账单

# 2. 非必要服务停机
sudo shutdown -h now   # 停机（不收费计算资源）
# 注意：磁盘仍收费

# 3. 使用竞价实例（便宜 50-90%）
# 缺点：可能被回收，适合可中断的任务

# 4. GPU 训练完成后释放
# AutoDL 等平台：训练完关机，按秒计费
```

**安全配置**：

```bash
# === 云服务器安全配置 ===

# 1. 禁用密码登录，只用 SSH 密钥
sudo nano /etc/ssh/sshd_config
# 设置: PasswordAuthentication no
sudo systemctl restart sshd

# 2. 配置防火墙（只开放必要端口）
sudo ufw allow 22      # SSH
sudo ufw allow 80      # HTTP
sudo ufw allow 443     # HTTPS
sudo ufw enable

# 3. 定期更新系统
sudo apt update && sudo apt upgrade -y

# 4. 敏感信息不要硬编码
# 使用环境变量或云服务商的密钥管理服务
```

**远程开发体验优化**：

```bash
# === VS Code 远程开发 ===

# 1. 安装 Remote-SSH 扩展

# 2. 配置 SSH 连接
# ~/.ssh/config
Host myserver
    HostName your-server-ip
    User username
    IdentityFile ~/.ssh/id_rsa

# 3. VS Code 连接远程
# Ctrl+Shift+P → Remote-SSH: Connect to Host → myserver

# 4. 远程开发体验接近本地
# - 代码在服务器上
# - 终端在服务器上
# - 但编辑在本地 VS Code
```

---

## 8.4 实验室/共享服务器开发：免费但需遵守规则

### 特点

- **多用户共享**：一台服务器多人使用，资源竞争
- **权限受限**：通常只有普通用户权限，不能装系统包
- **环境固定**：系统环境不能改，只能用已有的或自己安装
- **存储可能被清理**：临时文件可能被定期删除

### 共享服务器注意事项

**权限问题**：

```bash
# === 无 root 权限时的解决方案 ===

# 问题：想安装系统包但无权限
sudo apt install something  # 失败：Permission denied

# 解决方案 1：用 conda 安装（无需 root）
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh -p ~/miniconda3
source ~/miniconda3/bin/activate
conda install package-name

# 解决方案 2：从源码编译安装到用户目录
./configure --prefix=$HOME/.local
make && make install

# 解决方案 3：用 pip 的 --user 参数
pip install --user package-name
```

**资源竞争**：

```bash
# === 共享服务器资源使用规范 ===

# 1. 查看服务器资源使用情况
htop                    # 查看 CPU、内存使用
nvidia-smi              # 查看 GPU 使用（如果有）
df -h                   # 查看磁盘使用

# 2. 避免独占资源
# 训练时限制 CPU/内存使用
taskset -c 0-3 python train.py  # 只用前 4 个 CPU 核心

# 3. GPU 使用规范
# 查看哪些 GPU 空闲
nvidia-smi
# 使用指定 GPU
CUDA_VISIBLE_DEVICES=0 python train.py  # 只用 GPU 0

# 4. 后台运行任务
nohup python train.py > train.log 2>&1 &
# 或用 tmux/screen
tmux new -s training
python train.py
# Ctrl+B D 分离会话
tmux attach -t training  # 重新连接
```

**存储管理**：

```bash
# === 共享服务器存储注意事项 ===

# 1. 查看磁盘配额
quota -s

# 2. 查看自己占用空间
du -sh ~
du -sh ~/.cache
du -sh ~/projects/*

# 3. 清理不需要的文件
rm -rf ~/.cache/pip       # pip 缓存
rm -rf ~/.cache/huggingface  # HuggingFace 缓存（可能很大）

# 4. 大文件放共享存储或申请额外空间
# 不要在 home 目录放数据集
```

**环境隔离**：

```bash
# === 共享服务器环境隔离方案 ===

# 方案 1：conda 环境（推荐）
conda create -n myproject python=3.10
conda activate myproject
conda install -c conda-forge package-name

# 方案 2：virtualenv
python3 -m venv ~/venvs/myproject
source ~/venvs/myproject/bin/activate

# 方案 3：uv（更快）
curl -LsSf https://astral.sh/uv/install.sh | sh
uv venv
source .venv/bin/activate

# 重要：不要污染系统环境
# ❌ pip install package（影响所有用户）
# ✅ pip install --user package（只影响自己）
# ✅ 用虚拟环境（完全隔离）
```

### 共享服务器礼仪

| 规则 | 原因 |
|------|------|
| 不占用所有 GPU | 其他人也需要用 |
| 不跑无限循环占用 CPU | 影响其他人使用 |
| 定期清理临时文件 | 磁盘空间有限 |
| 不修改公共配置 | 影响所有人 |
| 大任务在夜间跑 | 白天资源紧张 |
| 用 tmux/screen 管理会话 | 断开连接不会中断任务 |

---

## 8.5 环境迁移：从本地到服务器

代码在本地开发后，需要部署到云服务器或实验室服务器。理解环境差异，才能避免"本地能跑，服务器跑不了"。

### 常见迁移问题

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 依赖版本不同 | 本地和服务器的包版本不一致 | 用锁文件（uv.lock / requirements.txt） |
| Python 版本不同 | 系统 Python 版本差异 | 用 pyenv 或 conda 管理版本 |
| 路径分隔符不同 | Windows 本地用 `\`，Linux 服务器用 `/` | 代码中用 `os.path.join()` 或 `pathlib` |
| 编码不同 | Windows 可能用 GBK，Linux 用 UTF-8 | 统一 UTF-8 |
| 端口被占用 | 服务器上端口被其他服务占用 | 修改配置或检查端口占用 |
| 权限不足 | 云服务器可能用非 root 用户运行 | 检查文件权限，用 `chmod` 修正 |
| 环境变量缺失 | .env 文件未同步或未加载 | 确保 .env 存在且正确加载 |

### 迁移检查清单

```bash
# === 部署前检查清单 ===

# 1. 确认 Python 版本
python --version
# 或
python3 --version

# 2. 锁定依赖版本
uv lock                    # 生成 uv.lock
# 或
pip freeze > requirements.txt

# 3. 检查环境变量
cat .env.example           # 确保模板完整
# 实际 .env 不提交，需要手动复制到服务器

# 4. 检查配置文件
# 确认数据库连接、API 地址等配置正确

# 5. 本地测试
uv run pytest tests/
uv run python run_server.py
curl http://localhost:8081/health

# 6. 打包代码
git status                 # 确认无未提交的改动
git push origin main       # 推送到远程仓库
```

```bash
# === 服务器部署流程 ===

# 1. 克隆代码
git clone https://github.com/user/project.git
cd project

# 2. 创建虚拟环境
uv sync

# 3. 配置环境变量
cp .env.example .env
nano .env  # 填入真实配置

# 4. 运行测试
uv run pytest tests/

# 5. 启动服务
uv run python run_server.py

# 6. 验证
curl http://localhost:8081/health
```

### 使用 Docker 简化迁移

Docker 可以避免大部分环境差异问题。

```bash
# === Docker 部署流程 ===

# 本地：
# 1. 编写 Dockerfile
# 2. 测试构建
docker build -t myapp .
docker run -p 8081:8000 myapp

# 3. 推送到镜像仓库
docker tag myapp registry.example.com/myapp:v1
docker push registry.example.com/myapp:v1

# 服务器：
# 1. 拉取镜像
docker pull registry.example.com/myapp:v1

# 2. 运行容器
docker run -d -p 80:8000 --name myapp \
  -e DATABASE_URL=... \
  registry.example.com/myapp:v1

# 3. 验证
curl http://localhost/health
```

---

## 8.6 关键认知

### 三个核心原则

1. **环境一致性是关键**：用 Docker 或锁文件确保开发=部署
2. **权限决定方案**：无 root 权限时用 conda/uv，有 root 可自由配置
3. **成本与便利权衡**：本地免费但有限，云服务付费但灵活

### 环境选择决策树

```
需要长期运行服务？
├─ 是 → 需要公网访问？
│       ├─ 是 → 用云服务器
│       └─ 否 → 实验室服务器（如果有）或本地
└─ 否 → 需要 GPU？
        ├─ 是 → 本地有 GPU？
        │       ├─ 是 → 本地开发
        │       └─ 否 → 云 GPU 或实验室服务器
        └─ 否 → 本地开发
```

### 常见陷阱速查

| 环境 | 常见陷阱 | 应对 |
|------|----------|------|
| 本地 | 环境与生产不一致 | 用 Docker |
| 云服务器 | 成本超支 | 设置预算告警，及时释放资源 |
| 共享服务器 | 权限不足 | 用 conda/uv 安装到用户目录 |
| 迁移时 | 依赖版本不一致 | 用锁文件 |
| 迁移时 | 配置缺失 | 确保 .env 同步 |

---

> **结语**：理解不同环境的差异，选择合适的工具和策略，才能在不同场景下高效开发。