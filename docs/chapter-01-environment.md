# 第一章：地基——环境隔离与依赖管理

> **核心哲学**：确保"在我的电脑上能跑，在别人的电脑上也能跑"

---

## 1.1 愿望清单与确定事实

### 原理：声明的两种形态

在 Python 生态中，依赖声明存在两种形态：

| 形态 | 文件 | 本质 | 版本约束 |
|------|------|------|----------|
| **愿望清单** | `pyproject.toml` | 声明"我需要什么" | 宽松范围（`>=`, `~=`） |
| **确定事实** | `uv.lock` / `poetry.lock` | 声明"我实际装了什么" | 精确锁定（`==`） |

**为什么需要两份文件？**

```
开发者 A 的机器：numpy 2.0.0（刚发布）
开发者 B 的机器：numpy 1.26.4（稳定版）

如果只有 pyproject.toml 声明 numpy>=1.20：
→ A 和 B 装的是不同版本
→ 可能有 API 差异
→ "在我机器上能跑"的噩梦开始
```

**锁文件的作用**：将"愿望"冻结为"事实"，确保所有环境完全一致。

---

### 实战：版本约束符号

```toml
# pyproject.toml

[project]
dependencies = [
    "requests>=2.28.0",     # 至少 2.28.0，允许 3.0.0（可能有破坏性变更）
    "numpy~=1.26.0",        # 兼容 1.26.x，但不允许 1.27.0
    "pandas==2.0.3",        # 必须精确版本（最严格，一般不推荐）
    "flask>=2.0,<3.0",      # 范围约束：2.x 系列
]
```

**约束力度对比**：

| 符号 | 含义 | 风险 | 推荐场景 |
|------|------|------|----------|
| `>=` | 最低版本 | 高（可能引入破坏性变更） | 内部工具、快速原型 |
| `~=` | 兼容版本 | 中（小版本内稳定） | **推荐默认** |
| `==` | 精确版本 | 低（但依赖树可能冲突） | 生产环境、关键依赖 |
| `>=x.y,<z.0` | 范围约束 | 中低 | 需要排除大版本 |

---

## 1.2 现代包管理引擎：uv

### 原理：为什么选择 uv？

传统 pip 的痛点：
- **慢**：串行下载，无缓存复用
- **依赖解析差**：容易陷入冲突死循环
- **无锁文件**：需要配合 pip-tools 等工具

uv 的优势：
- **Rust 实现**：10-100x 速度提升
- **内置锁文件**：自动生成 `uv.lock`
- **虚拟环境集成**：`uv run` 自动激活环境
- **兼容性好**：完全兼容 pip 命令

---

### 实战：核心命令

#### 1. 同步环境：`uv sync`

```bash
# 根据锁文件精确安装（推荐生产环境）
uv sync

# 只安装生产依赖，排除开发依赖
uv sync --no-dev

# 重新生成锁文件（当 pyproject.toml 改动后）
uv lock
uv sync
```

**工作流程图**：

```
pyproject.toml (愿望)
        │
        ▼
    uv lock  ──────► uv.lock (事实)
        │
        ▼
    uv sync ──────► .venv/ (环境)
```

---

#### 2. 隔离执行：`uv run`

```bash
# 在隔离环境中运行脚本（推荐所有 Python 执行）
uv run python train.py

# 运行测试
uv run pytest tests/

# 运行模块
uv run python -m http.server 8000
```

**为什么不用 `python script.py`？**

| 命令 | 环境来源 | 问题 |
|------|----------|------|
| `python script.py` | 系统 Python / 当前激活的 venv | 可能用错环境 |
| `uv run python script.py` | 项目 `.venv` | **确保一致性** |

---

#### 3. 全局工具：`uv tool`

```bash
# 安装全局命令行工具（不影响项目环境）
uv tool install ruff
uv tool install black
uv tool install httpie

# 运行一次性工具（无需安装）
uv tool run ruff check .

# 查看已安装工具
uv tool list
```

**使用场景**：
- 代码格式化工具（black, ruff）
- 代码检查工具（mypy, pylint）
- 实用工具（httpie, jq）

---

## 1.3 传统基石：pip

### 原理：理解 pip 的角色

pip 是 Python 官方的包管理器，虽然 uv 在速度上碾压它，但理解 pip 依然是必要的：

1. **兼容性要求**：很多 CI/CD 脚本、Dockerfile 仍使用 pip
2. **原理理解**：uv 本质上是 pip 的优化实现，底层逻辑一致
3. **PyPI 生态**：所有工具都从 PyPI（Python Package Index）获取包

---

### 实战：pip 关键命令

```bash
# 安装包
pip install requests           # 最新版
pip install requests==2.28.0   # 指定版本
pip install -r requirements.txt  # 从文件安装

# 查看已安装
pip list
pip show requests              # 包详情

# 导出依赖（传统方式，无锁文件）
pip freeze > requirements.txt

# 卸载
pip uninstall requests
```

---

### 进阶：requirements.txt vs pyproject.toml

| 维度 | requirements.txt | pyproject.toml |
|------|------------------|----------------|
| 标准 | 非官方约定 | PEP 517/518 官方标准 |
| 元数据 | 无 | 支持项目名称、版本、描述等 |
| 依赖分组 | 需要多个文件 | 原生支持 `[project.optional-dependencies]` |
| 构建系统 | 不涉及 | 定义构建后端（setuptools, hatch, flit） |
| 现代 Python | 逐渐淘汰 | **推荐** |

---

## 1.4 实战案例：完整工作流

### 项目依赖结构

```
my-project/
├── pyproject.toml      # 愿望清单
├── uv.lock             # 锁定事实（自动生成）
└── .venv/              # 虚拟环境（uv sync 生成）
```

### 常见工作流

```bash
# 1. 克隆项目后首次设置
git clone https://github.com/user/my-project.git
cd my-project
uv sync                    # 创建 .venv 并安装所有依赖

# 2. 添加新依赖
uv add fastapi             # 自动更新 pyproject.toml 和 uv.lock

# 3. 添加开发依赖
uv add --dev pytest ruff   # 添加到 [project.optional-dependencies]

# 4. 运行项目
uv run python main.py

# 5. 运行测试
uv run pytest tests/

# 6. 进入虚拟环境 Shell（可选，不推荐）
source .venv/bin/activate  # Linux/macOS
.venv\Scripts\activate     # Windows
```

---

## 1.5 关键认知

### 三个核心原则

1. **环境隔离是底线**：永远不要用系统 Python 装项目依赖
2. **锁文件是保险**：`uv.lock` 提交到 Git，确保团队一致性
3. **uv run 是习惯**：所有 Python 命令前加 `uv run`

### 一个命令检验环境健康

```bash
# 检查当前使用的 Python 是否在项目虚拟环境中
which python              # Linux/macOS
where python              # Windows

# 正确输出示例：
# /home/user/project/.venv/bin/python

# 错误输出示例：
# /usr/bin/python          # 这是系统 Python！
```

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 忘记 `uv sync` | ModuleNotFoundError | 克隆后立即执行 `uv sync` |
| 用系统 Python | 版本冲突、污染系统 | 始终用 `uv run` |
| 忽略锁文件 | "在我机器上能跑" | 将 `uv.lock` 提交到 Git |
| 手动编辑锁文件 | 依赖解析失败 | 锁文件由工具管理，不要手动改 |

---

> **下一章预告**：有了环境作为地基，我们将学习如何通过终端（CLI）与操作系统高效对话，包括管道符、重定向、grep 搜索等核心技能。