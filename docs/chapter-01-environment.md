# 第一章：地基——环境隔离与依赖管理

> **核心哲学**：确保"在我的电脑上能跑，在别人的电脑上也能跑"

---

## 1.0 开发环境基础：Windows、Linux 与 WSL

在深入环境隔离和依赖管理之前，先理解不同操作系统环境的差异。这是后续所有环境管理决策的基础认知。

### 三种开发环境对比

开发者常用的三种环境：Windows 本地、Linux 本地、WSL（Windows Subsystem for Linux）。

| 维度 | Windows 本地 | Linux 本地 | WSL（Windows 中运行 Linux） |
|------|-------------|-----------|---------------------------|
| 操作系统 | Windows 10/11 | Ubuntu/Debian/CentOS 等 | Linux 子系统，运行在 Windows 中 |
| 命令行 | PowerShell / CMD | Bash / Zsh | Bash（Linux 命令） |
| 包管理 | 无统一方案 | apt/yum/dnf | apt（Linux 包管理器） |
| 文件系统 | NTFS (C:\, D:\) | ext4 (/home, /etc) | 混合（/mnt/c/ 访问 Windows） |
| 路径格式 | 反斜杠 `\` | 正斜杠 `/` | 正斜杠 `/` |
| 行尾符 | CRLF (`\r\n`) | LF (`\n`) | LF (`\n`) |
| 开发体验 | 需额外配置 | 原生友好 | 接近 Linux 体验 |

**环境选择建议**：

| 场景 | 推荐环境 | 原因 |
|------|----------|------|
| 日常开发（Python/Node） | **WSL** | 接近 Linux 体验，工具链完善 |
| 前端开发（需要图形调试） | Windows 本地 | 浏览器、图形工具在 Windows 更方便 |
| 后端/运维/DevOps | **Linux 本地** 或 WSL | 生产环境通常是 Linux，开发环境需匹配 |
| 学习 Linux | WSL | 无需额外机器或虚拟机 |

### WSL：Windows 中的 Linux 子系统

WSL 让你在 Windows 中直接运行 Linux，无需虚拟机。这是 Windows 开发者的最佳选择：既能用 Windows 的图形工具，又能获得 Linux 的开发体验。

**WSL 核心特点**：

```
Windows 主系统
├── WSL 子系统（Ubuntu/Debian 等）
│   ├── Linux 命令行（bash）
│   ├── Linux 文件系统（/home, /etc）
│   ├── Linux 包管理器（apt）
│   └── 与 Windows 文件互访（/mnt/c/）
└── Windows 本地应用（VS Code、浏览器等）
```

**WSL 与 Windows 的文件互访**：

| 方向 | 路径 | 示例 |
|------|------|------|
| WSL 访问 Windows | `/mnt/c/`, `/mnt/d/` | `/mnt/c/Users/你的用户名/` |
| Windows 访问 WSL | `\\wsl$\` | `\\wsl$\Ubuntu\home\user\` |

```bash
# === WSL 中访问 Windows 文件 ===
cd /mnt/c/Users/你的用户名/    # 进入 Windows 用户目录
ls /mnt/d/projects/            # 查看 D 盘项目

# === Windows 中访问 WSL 文件（在 PowerShell 中）===
cd \\wsl$\Ubuntu\home\user\    # 进入 WSL 用户目录
```

**推荐做法**：项目代码放在 WSL 文件系统中（`/home/user/projects/`），而非 Windows 文件系统（`/mnt/c/`）。原因：WSL 文件系统性能更好、行尾符一致、权限管理正确。

### IDE 中打开不同环境的区别

VS Code（或其他 IDE）可以打开两种位置的文件夹：
- **Windows 本地文件夹**：如 `C:\Users\xxx\projects\`
- **WSL 文件夹**：如 `/home/user/projects/`（通过 WSL 扩展）

两种方式的关键差异：

| 维度 | 打开 Windows 本地文件夹 | 打开 WSL 文件夹 |
|------|------------------------|----------------|
| 终端类型 | PowerShell / CMD | Bash |
| 命令语法 | Windows 命令（`dir`, `copy`） | Linux 命令（`ls`, `cp`） |
| 路径格式 | `C:\Users\xxx\` | `/home/user/` |
| 行尾符 | 默认 CRLF（需手动配置 LF） | 默认 LF |
| 文件权限 | Windows ACL | Linux 权限（chmod） |
| 运行 Python | 需 Windows Python | 直接 `uv run python` |
| 运行 Shell 脚本 | 需 Git Bash 或转换 | 直接 `bash script.sh` |
| 性能 | 一般 | WSL 文件系统中更快 |

**如何区分当前打开的是哪种环境**：

1. **看终端**：终端标签显示 `WSL: Ubuntu` 或 `PowerShell`
2. **看左下角**：VS Code 左下角显示 `WSL: Ubuntu` 绿色指示器
3. **看路径**：路径开头是 `/home/`（WSL）还是 `C:\`（Windows）

```bash
# === 快速判断当前环境 ===
# 在终端中运行：
pwd              # /home/user/... → WSL
                 # /mnt/c/Users/... → Windows 目录（但在 WSL 中）
                 # C:\Users\... → Windows

whoami           # 显示用户名，WSL 和 Windows 用户名可能不同
uname -a         # Linux 信息 → WSL
                 # 报错或无输出 → Windows
```

**开发优劣对比**：

| 场景 | Windows 本地 | WSL |
|------|-------------|-----|
| 运行 Python 项目 | 需安装 Windows Python | 原生 Linux Python，更稳定 |
| 使用 uv/pip | 需额外安装 | 直接可用 |
| Shell 脚本执行 | 需 Git Bash 或 WSL | 直接执行 |
| Git 操作 | Git for Windows | Linux Git，行为一致 |
| Docker 开发 | Docker Desktop | Docker Desktop（WSL 2 后端） |
| 文件权限问题 | 常见（权限差异） | 少见（Linux 权限模型） |
| 行尾符问题 | 常见（CRLF/LF 混合） | 少见（统一 LF） |
| 图形工具调试 | 方便（浏览器在 Windows） | 方便（WSLg 或 Windows 侧） |

**结论**：大多数后端/Python/Node 开发推荐在 WSL 环境中进行，前端需要频繁浏览器调试可在 Windows 本地，但项目代码仍建议放在 WSL。

### Bash 与 PowerShell/CMD 的区别

Bash 是 Linux/macOS/WSL 的标准命令行，PowerShell 和 CMD 是 Windows 的命令行。语法差异很大。

**常用命令对比**：

| 功能 | Bash | PowerShell | CMD |
|------|------|------------|-----|
| 列出文件 | `ls` 或 `ls -la` | `Get-ChildItem` 或 `dir` | `dir` |
| 切换目录 | `cd /home/user/` | `cd C:\Users\` | `cd C:\Users\` |
| 查看文件内容 | `cat file.txt` | `Get-Content file.txt` | `type file.txt` |
| 复制文件 | `cp src dest` | `Copy-Item src dest` | `copy src dest` |
| 移动文件 | `mv src dest` | `Move-Item src dest` | `move src dest` |
| 删除文件 | `rm file.txt` | `Remove-Item file.txt` | `del file.txt` |
| 创建目录 | `mkdir dir/` | `mkdir dir` 或 `New-Item` | `mkdir dir` |
| 查找文本 | `grep "pattern" file` | `Select-String "pattern"` | 无原生命令 |
| 管道 | `|`（传递输出） | `|`（传递对象） | `|`（传递文本） |
| 环境变量 | `$VAR` 或 `$HOME` | `$env:VAR` | `%VAR%` |
| 当前路径 | `pwd` | `pwd` 或 `Get-Location` | `cd`（无参数） |

**Bash 管道威力**：

```bash
# === Bash 管道组合示例 ===
# 查找所有 .py 文件中包含 "import" 的行
find . -name "*.py" | xargs grep "import"

# 查看最近修改的 5 个文件
ls -lt | head -5

# 统计代码行数
find . -name "*.py" | xargs wc -l | tail -1

# 实时查看日志并过滤
tail -f app.log | grep "ERROR"
```

**建议**：无论在 WSL 还是 Linux，学习 Bash 是必要的。Bash 管道和脚本能力远强于 PowerShell/CMD。

### VS Code 界面功能详解

VS Code（或其他现代 IDE）界面复杂，理解每个面板的作用才能高效使用。

#### 底部面板：问题、输出、调试控制台、终端、端口

| 面板 | 作用 | 使用场景 |
|------|------|----------|
| **问题** | 显示代码错误、警告 | 查看语法错误、linter 提示、类型检查结果 |
| **输出** | 显示扩展、任务的输出日志 | 查看构建日志、扩展运行日志、任务执行结果 |
| **调试控制台** | 调试时交互执行代码 | 断点调试时，执行表达式、查看变量、调用函数 |
| **终端** | 命令行交互 | 运行命令、执行脚本、Git 操作、启动服务 |
| **端口** | 显示正在监听的端口 | 查看服务端口、自动转发端口（远程开发） |

**面板使用场景**：

```
┌─────────────────────────────────────────────────────────────┐
│  问题（Problems）                                            │
│  ├─ 代码编写时：语法错误实时显示                              │
│  ├─ Linter 运行后：代码规范警告                               │
│  ├─ 类型检查后：类型错误提示                                  │
│  └─ 双击错误行：跳转到对应代码位置                            │
├─────────────────────────────────────────────────────────────┤
│  输出（Output）                                              │
│  ├─ 执行任务时：显示任务执行日志                              │
│  ├─ 扩展运行时：显示扩展内部日志                              │
│  ├─ Git 操作时：显示 Git 命令输出                             │
│  └─ 与终端区别：输出是被动的，终端是主动交互                   │
├─────────────────────────────────────────────────────────────┤
│  调试控制台（Debug Console）                                 │
│  ├─ 仅在调试时使用：断点暂停时才能交互                        │
│  ├─ 执行表达式：> variable_name                              │
│  ├─ 调用函数：> some_function()                              │
│  ├─ 查看复杂对象：> JSON.stringify(obj)                       │
│  └─ 与终端区别：在当前断点上下文中执行                        │
├─────────────────────────────────────────────────────────────┤
│  终端（Terminal）                                            │
│  ├─ 日常命令：ls, cd, cat, grep                              │
│  ├─ 运行项目：uv run python main.py                          │
│  ├─ Git 操作：git status, git add, git commit                │
│  ├─ 启动服务：uv run python run_server.py                    │
│  ├─ 测试运行：uv run pytest tests/                           │
│  └─ 可开多个：不同任务用不同终端                              │
├─────────────────────────────────────────────────────────────┤
│  端口（Ports）                                               │
│  ├─ 显示服务端口：localhost:8081 → Python 服务              │
│  ├─ 显示前端端口：localhost:3001 → Next.js 服务              │
│  ├─ 点击端口：在浏览器打开                                    │
│  └─ 远程开发时：自动转发远程端口到本地                        │
└─────────────────────────────────────────────────────────────┘
```

**终端快捷操作**：

```bash
# === VS Code 终端快捷键 ===
Ctrl + `         # 打开/关闭终端
Ctrl + Shift + ` # 新建终端
Ctrl + PageUp    # 切换到上一个终端
Ctrl + PageDown  # 切换到下一个终端
Ctrl + C         # 终止当前命令（或复制，取决于焦点）
Ctrl + L         # 清屏
```

#### 左侧边栏：文件、大纲、时间线

| 面板 | 作用 | 使用场景 |
|------|------|----------|
| **资源管理器（文件栏）** | 项目文件树 | 浏览、打开、创建、删除文件 |
| **大纲** | 当前文件的代码结构 | 快速跳转到函数、类、变量定义 |
| **时间线** | 文件的 Git 历史 | 查看文件修改历史、对比版本差异 |

**大纲详解**：

```
大纲（Outline）显示当前打开文件的代码结构：

┌─────────────────────────────────┐
│  📄 main.py                     │
│  ├─ 📦 class UserService        │  ← 点击跳转到类定义
│  │   ├─ ⚙️ __init__             │  ← 点击跳转到方法
│  │   ├─ ⚙️ get_user             │
│  │   └─ ⚙️ save_user            │
│  ├─ 🔧 def main()               │  ← 点击跳转到函数
│  ├─ 📦 class Config             │
│  └─ 🔵 STATUS_ACTIVE            │  ← 点击跳转到常量
└─────────────────────────────────┘

符号类型：
📦 类 / 🔧 函数 / 🔵 变量 / 📝 注释块
```

**时间线详解**：

```
时间线（Timeline）显示当前文件的 Git 提交历史：

┌─────────────────────────────────┐
│  📄 main.py                     │
│  ├─ 2024-03-29 14:30            │
│  │   "添加用户认证逻辑"          │  ← 点击查看该版本的文件内容
│  ├─ 2024-03-28 10:15            │
│  │   "重构用户服务"              │  ← 右键可对比与当前版本差异
│  ├─ 2024-03-25 09:00            │
│  │   "初始化项目结构"            │
└─────────────────────────────────┘

用途：
- 查看文件是谁改的、什么时候改的、改了什么
- 找回被删除的代码片段
- 对比当前版本和历史版本差异
```

#### 右下角状态栏：文件信息

状态栏右下角显示当前文件的关键信息：

```
┌───────────────────────────────────────────────────────────────┐
│  UTF-8  │  LF  │  Python  │  空格: 4  │  Pylance  │  行 42, 列 8 │
└───────────────────────────────────────────────────────────────┘
```

| 信息 | 含义 | 常见问题 |
|------|------|----------|
| **UTF-8** | 文件编码（字符集） | 编码错误会导致中文乱码、特殊字符丢失 |
| **LF** | 行尾符类型 | LF/CRLF 混合会导致 Git diff 噪音、脚本执行失败 |
| **Python** | 文件类型/语言模式 | 决定语法高亮、智能补全、linter 使用哪种规则 |
| **空格: 4** | 缩进方式 | 混合空格和 Tab会导致语法错误 |
| **Pylance** | 语言服务器 | 提供智能补全、类型检查、错误提示 |
| **行 42, 列 8** | 光标位置 | 快速定位代码位置 |

**编码（UTF-8 vs GBK）详解**：

```
编码决定计算机如何理解文字：

UTF-8（推荐）
├── 全球通用，支持所有语言
├── Python 3 默认编码
├── Linux/macOS 默认编码
└── 几乎所有现代项目都用 UTF-8

GBK/GB2312（Windows 中文旧版）
├── 仅支持中文
├── Windows 中文版历史默认
├── 容易导致乱码问题
└── 现代项目应避免使用

常见问题：
├── 中文乱码：文件编码与工具解码不一致
├── Git 显示差异：实际无改动，只是编码不同
├── 脚本执行失败：编码错误导致 Python 解析失败

解决方案：
├── 统一使用 UTF-8
├── VS Code：右下角点击编码 → Reopen with Encoding → UTF-8
├── VS Code：保存时 → Save with Encoding → UTF-8
```

**行尾符（LF vs CRLF）详解**：

```
行尾符决定每行结束的标记：

LF（Line Feed，\n）
├── Linux/macOS/WSL 标准
├── 一个字符
├── Git、Shell 脚本推荐
└── 现代开发推荐统一使用 LF

CRLF（Carriage Return + Line Feed，\r\n）
├── Windows 标准
├── 两个字符
├── Windows 记事本、某些 Windows 工具默认
└── 混合使用会导致问题

常见问题：
├── Git diff 显示大量改动（实际只是换行符）
├── Shell 软件执行失败（bash 不认 \r）
├── Python 报错：SyntaxError 或解析异常
├── 多人协作时互相"污染"文件

解决方案：
├── 统一使用 LF
├── VS Code：右下角点击 LF/CRLF → 切换
├── Git 配置：git config --global core.autocrlf input
├── .gitattributes 文件：*.py text eol=lf
```

**VS Code 设置统一 LF 和 UTF-8**：

```json
// settings.json
{
    "files.encoding": "utf8",
    "files.eol": "\n",
    "files.insertFinalNewline": true
}
```

### 开发环境统一最佳实践

多人协作时，环境差异会导致大量问题。统一标准能减少协作摩擦。

| 项目 | 统一标准 | 配置方式 |
|------|----------|----------|
| 编码 | UTF-8 | VS Code 设置、EditorConfig |
| 行尾符 | LF | VS Code 设置、.gitattributes |
| 缩进 | 空格 4（Python） | VS Code 设置、EditorConfig |
| Python 版本 | pyproject.toml 声明 | `requires-python = ">=3.10"` |
| 包管理 | uv | pyproject.toml + uv.lock |
| 命令环境 | WSL 或 Linux | 团队统一选择 |

**EditorConfig 跨编辑器统一**：

```ini
# .editorconfig（提交到 Git，所有编辑器遵守）
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.py]
indent_style = space
indent_size = 4

[*.{js,ts,json,yaml,yml}]
indent_style = space
indent_size = 2

[Makefile]
indent_style = tab
```

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