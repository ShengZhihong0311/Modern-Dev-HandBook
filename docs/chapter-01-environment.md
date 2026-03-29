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

## 1.5 关键认知

三个核心原则：

1. **环境隔离是底线**：永远不要用系统 Python 装项目依赖
2. **锁文件是保险**：`uv.lock` 提交到 Git，确保团队一致性
3. **uv run 是习惯**：所有 Python 命令前加 `uv run`

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 忘记 `uv sync` | ModuleNotFoundError | 克隆后立即执行 |
| 用系统 Python | 版本冲突 | 始终用 `uv run` |
| 忽略锁文件 | "在我机器上能跑" | 将 `uv.lock` 提交 Git |
| 手动编辑锁文件 | 依赖解析失败 | 锁文件由工具管理 |

---

> **下一章**：有了环境作为地基，我们将学习如何通过终端与操作系统高效对话。