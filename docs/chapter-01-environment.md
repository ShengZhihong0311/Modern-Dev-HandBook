# 第一章：地基——环境隔离与依赖管理

> **核心哲学**：确保"在我的电脑上能跑，在别人的电脑上也能跑"

---

## 1.1 愿望与事实：两种声明形态

在 Python 生态中，依赖声明存在两种形态：`pyproject.toml` 是"愿望清单"，声明你需要什么；`uv.lock` 是"确定事实"，记录你实际装了什么。

为什么要两份文件？想象这个场景：开发者 A 的机器装了 numpy 2.0.0（刚发布），开发者 B 的机器装了 numpy 1.26.4（稳定版）。如果 `pyproject.toml` 只声明 `numpy>=1.20`，两人装的版本不同，API 可能差异，"在我机器上能跑"的噩梦就此开始。锁文件的作用，就是把"愿望"冻结为"事实"，确保所有环境完全一致。

### 版本约束符号对比

| 符号 | 含义 | 推荐场景 |
|------|------|----------|
| `>=` | 最低版本，允许更高版本 | 内部工具、快速原型 |
| `~=` | 兼容版本，锁定小版本 | **推荐默认** |
| `==` | 精确版本 | 生产环境、关键依赖 |
| `>=x.y,<z.0` | 范围约束 | 排除某个大版本 |

### 实战命令

```bash
# 查看当前依赖声明
cat pyproject.toml

# 查看锁定的精确版本
cat uv.lock | grep "numpy"

# 添加新依赖（自动更新两份文件）
uv add requests~=2.28.0

# 添加开发依赖
uv add --dev pytest~=8.0
```

---

## 1.2 现代包管理：uv

传统 pip 有三个痛点：慢（串行下载）、依赖解析差（容易冲突）、无锁文件（需要 pip-tools）。uv 用 Rust 重写，速度提升 10-100x，内置锁文件，`uv run` 自动激活环境。

uv 的核心工作流很简单：`pyproject.toml`（愿望）经过 `uv lock` 生成 `uv.lock`（事实），再通过 `uv sync` 创建 `.venv`（环境）。

### 实战命令

```bash
# 首次克隆项目后的标准流程
git clone https://github.com/user/project.git
cd project
uv sync                    # 创建虚拟环境并安装依赖

# 运行任何 Python 代码（推荐始终用 uv run）
uv run python main.py
uv run pytest tests/
uv run python -m http.server 8000

# 全局安装命令行工具（不影响项目环境）
uv tool install ruff
uv tool run black .

# 添加/移除依赖
uv add fastapi
uv remove fastapi
```

### pip 与 uv 对比

| 操作 | pip | uv |
|------|-----|-----|
| 安装依赖 | `pip install -r requirements.txt` | `uv sync` |
| 运行脚本 | `python script.py`（需手动激活环境） | `uv run python script.py` |
| 锁文件 | 无（需 pip-tools） | 自动生成 `uv.lock` |
| 速度 | 慢（串行） | 快（Rust 并行） |

---

## 1.3 传统基石：pip 与 PyPI

虽然 uv 更快，但理解 pip 依然必要：很多 CI/CD 脚本和 Dockerfile 仍用 pip；uv 底层逻辑与 pip 一致；所有工具都从 PyPI（Python Package Index）获取包。

pip 的核心命令依然要掌握，尤其是在维护旧项目或排查问题时。

### 实战命令

```bash
# 安装包
pip install requests
pip install requests==2.28.0
pip install -r requirements.txt

# 查看已安装
pip list
pip show requests

# 导出依赖（传统方式）
pip freeze > requirements.txt

# 卸载
pip uninstall requests
```

### requirements.txt vs pyproject.toml

| 维度 | requirements.txt | pyproject.toml |
|------|------------------|----------------|
| 标准 | 非官方约定 | PEP 517/518 官方标准 |
| 元数据 | 无 | 支持名称、版本、描述 |
| 依赖分组 | 需要多个文件 | 原生支持 optional-dependencies |
| 现代 Python | 逐渐淘汰 | **推荐** |

---

## 1.4 完整工作流案例

一个项目的标准依赖结构：`pyproject.toml`（愿望）、`uv.lock`（事实）、`.venv`（环境）。三者配合，确保"在我机器上能跑，在别人机器上也能跑"。

### 实战命令

```bash
# === 项目初始化 ===
uv sync                    # 首次设置

# === 添加依赖 ===
uv add fastapi             # 生产依赖
uv add --dev pytest ruff   # 开发依赖

# === 运行项目 ===
uv run python main.py
uv run pytest tests/

# === 检查环境 ===
which python               # 应输出 .venv/bin/python
uv pip list                # 查看已安装包
```

---

## 1.5 环境变量与忽略文件：配置隔离

代码中不应该硬编码敏感信息（密码、API Key、数据库连接）。环境变量文件 `.env` 实现配置与代码分离，`.gitignore` 防止敏感文件进入版本控制。两者配合，保护项目安全。

### .env 文件规范

`.env` 存放环境特定配置：数据库连接、API 密钥、第三方服务凭证。核心原则：**绝不提交到 Git**，只提供 `.env.example` 模板。

**.env 内容分类**：

| 类别 | 示例 | 安全等级 |
|------|------|----------|
| 数据库配置 | `DATABASE_URL=postgres://...` | 高危（含密码） |
| API 密钥 | `OPENAI_API_KEY=sk-...` | 高危 |
| 第三方凭证 | `AWS_ACCESS_KEY_ID=...` | 高危 |
| 服务端口 | `PORT=8081` | 低危 |
| 调试开关 | `DEBUG=true` | 低危 |
| 功能开关 | `ENABLE_CACHE=true` | 低危 |

**命名规范**：

```bash
# === 好的命名 ===
DATABASE_URL                 # 服务名 + 功能
OPENAI_API_KEY              # 服务名 + 类型
REDIS_HOST                  # 服务名 + 属性
LOG_LEVEL                   # 功能名 + 属性

# === 坏的命名 ===
DB                          # 过短，含义不清
key                         # 不明确是哪个服务
password                    # 不明确属于谁
VAR1                        # 无意义
```

**.env.example 模板规范**：

```bash
# .env.example（提交到 Git，不含真实值）

# === 数据库配置 ===
DATABASE_URL=postgres://user:password@localhost:5432/dbname
DATABASE_POOL_SIZE=5

# === API 配置 ===
OPENAI_API_KEY=your-api-key-here
MODEL_PROVIDER=deepseek

# === 服务配置 ===
PORT=8081
DEBUG=false
LOG_LEVEL=INFO

# === 功能开关 ===
ENABLE_CACHE=true
CACHE_EXPIRE_SECONDS=3600
```

### .gitignore 文件规范

`.gitignore` 声明哪些文件不应进入版本控制。核心目标：防止敏感信息泄露、减少仓库体积、避免 IDE/系统差异文件干扰。

**必须忽略的内容**：

| 类别 | 必须忽略 | 原因 |
|------|----------|------|
| 环境变量 | `.env`, `.env.local`, `.env.*.local` | **高危**：含敏感信息 |
| 依赖目录 | `.venv/`, `node_modules/`, `__pycache__/` | 体积大，可重建 |
| IDE 配置 | `.idea/`, `.vscode/`, `*.swp` | 个人偏好，不共享 |
| 系统文件 | `.DS_Store`, `Thumbs.db` | 无意义 |
| 构建产物 | `dist/`, `build/`, `*.pyc` | 可重建 |
| 日志文件 | `*.log`, `logs/` | 运行时生成 |
| 临时文件 | `*.tmp`, `*.temp`, `.cache/` | 临时数据 |

**Python 项目 .gitignore 模板**：

```gitignore
# === 环境变量（最高优先级）===
.env
.env.local
.env.*.local

# === Python ===
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
.venv/
venv/
ENV/

# === 测试与覆盖 ===
.pytest_cache/
.coverage
htmlcov/
.tox/

# === IDE ===
.idea/
.vscode/
*.swp
*.swo
*~

# === 系统 ===
.DS_Store
Thumbs.db

# === 构建产物 ===
dist/
build/
*.egg-info/

# === 日志与临时 ===
*.log
logs/
.cache/
*.tmp

# === 本地开发脚本 ===
scripts/local/
```

### 实战命令

```bash
# === 验证 .gitignore ===
git check-ignore -v .env          # 检查是否被忽略
git check-ignore -v .venv/

# === 查看 .env 加载情况 ===
# Python (python-dotenv)
uv run python -c "from dotenv import load_dotenv; load_dotenv(); import os; print(os.getenv('DATABASE_URL'))"

# === 安全检查（确保 .env 未提交）===
git log --all --full-history -- ".env"  # 检查历史中是否有 .env
# 如果有，需要从历史中移除（高危操作，慎重）
```

---

## 1.6 关键认知

四个核心原则：

1. **环境隔离是底线**：永远不要用系统 Python 装项目依赖
2. **锁文件是保险**：`uv.lock` 提交到 Git，确保团队一致性
3. **uv run 是习惯**：所有 Python 命令前加 `uv run`
4. **.env 绝不提交**：敏感信息用环境变量，模板用 `.env.example`

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 忘记 `uv sync` | ModuleNotFoundError | 克隆后立即执行 |
| 用系统 Python | 版本冲突 | 始终用 `uv run` |
| 忽略锁文件 | "在我机器上能跑" | 将 `uv.lock` 提交 Git |
| 手动编辑锁文件 | 依赖解析失败 | 锁文件由工具管理 |
| 提交 .env 文件 | **安全事故** | `.gitignore` 忽略，只提交 `.env.example` |
| 硬编码敏感信息 | 密码泄露到 Git | 用环境变量替代 |
| .gitignore 缺失 | 仓库体积膨胀、IDE 配置冲突 | 使用标准模板 |

---

> **下一章**：有了环境作为地基，我们将学习如何通过终端与操作系统高效对话。