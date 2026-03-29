# 第四章：时空——Git 版本控制与协作

> **核心哲学**：管理代码的演进史，而不是单纯的备份

---

## 4.1 版本控制的本质

Git 不是备份工具，而是时间机器。每次 commit 是一个"快照"，记录了项目在某个时刻的完整状态。通过 commit 历史，你可以穿越到任意时刻、查看任意变更、回退到任意版本。

Git 的核心概念：仓库（repository）是项目的容器，commit 是时间节点，branch 是平行宇宙，merge 是宇宙交汇。理解这些概念，就能理解所有 Git 命令的本质。

### Git 核心概念对比

| 概念 | 本质 | 常用命令 |
|------|------|----------|
| 仓库 | 项目容器 | `git init`, `git clone` |
| commit | 时间快照 | `git commit` |
| branch | 平行开发线 | `git branch`, `git checkout` |
| merge | 分支合并 | `git merge` |
| remote | 远程仓库 | `git remote`, `git push`, `git pull` |

### 实战命令

```bash
# === 初始化仓库 ===
git init                               # 创建新仓库
git clone https://github.com/user/repo.git  # 克隆远程仓库

# === 查看状态 ===
git status                             # 当前状态：修改、暂存、未跟踪
git status -s                          # 简洁模式

# === 基础流程 ===
git add file.txt                       # 暂存单个文件
git add .                              # 暂存所有变更
git commit -m "feat: 添加新功能"       # 提交快照
git push origin main                   # 推送到远程
```

---

## 4.2 查看历史：穿越时空

`git log` 查看 commit 历史，`git show` 查看单个 commit 的详情，`git diff` 查看变更差异。这些命令是理解项目演进的窗口。

### 实战命令

```bash
# === 查看历史 ===
git log                                # 完整历史
git log --oneline                      # 简洁模式：一行一个 commit
git log --oneline -10                  # 最近 10 个 commit
git log --graph                        # 图形化显示分支结构
git log --oneline --graph --all        # 所有分支的图形化历史

# === 查看单个 commit ===
git show                               # 最新 commit 详情
git show abc123                        # 指定 commit 详情
git show abc123 --stat                 # 只看文件变更统计

# === 查看差异 ===
git diff                               # 工作区 vs 暂存区
git diff --staged                      # 暂存区 vs 最新 commit
git diff abc123 def456                 # 两个 commit 之间
git diff main feature                  # 两个分支之间

# === 查看文件历史 ===
git log --follow file.txt              # 文件的完整历史（含重命名）
git log -p file.txt                    # 文件每次变更的差异
```

---

## 4.3 分支管理：平行宇宙

分支是 Git 最强大的特性。每个分支是独立的开发线，可以并行开发不同功能，最后合并到主线。主分支（main/master）是稳定版本，功能分支（feature/*）是开发版本。

分支的本质：commit 的指针。创建分支只是创建新指针，不会复制文件。切换分支是移动 HEAD 指针到不同 commit。

### 分支策略对比

| 分支类型 | 名称约定 | 用途 |
|----------|----------|------|
| 主分支 | main / master | 生产环境代码 |
| 开发分支 | dev / develop | 集成测试 |
| 功能分支 | feature/* | 新功能开发 |
| 修复分支 | fix / bugfix | Bug 修复 |
| 发布分支 | release/* | 版本发布准备 |

### 实战命令

```bash
# === 创建分支 ===
git branch feature-login               # 创建分支
git checkout feature-login             # 切换分支
git checkout -b feature-login          # 创建并切换（推荐）

# === 查看分支 ===
git branch                             # 本地分支
git branch -a                          # 所有分支（含远程）
git branch -v                          # 分支 + 最新 commit

# === 合并分支 ===
git checkout main                      # 切回主分支
git merge feature-login                # 合并功能分支
git merge feature-login --no-ff        # 禁用快进合并（保留分支历史）

# === 删除分支 ===
git branch -d feature-login            # 删除已合并的分支
git branch -D feature-login            # 强制删除（未合并也删）

# === 推送分支 ===
git push origin feature-login          # 推送分支到远程
git push origin --delete feature-login # 删除远程分支
```

---

## 4.4 Merge vs Rebase：合并的两种哲学

merge 和 rebase 是整合分支的两种方式。merge 创建新 commit，保留分支历史；rebase 重写历史，使分支线性化。两者各有优劣，选择取决于团队偏好。

merge 的结果是多一个 commit，历史图有分叉。rebase 的结果是历史图是一条直线，但改变了 commit 的时间线。原则：公共分支用 merge，私有分支可用 rebase。

### Merge vs Rebase 对比

| 方式 | 历史形态 | 适用场景 | 风险 |
|------|----------|----------|------|
| merge | 分叉保留 | 公共分支、团队协作 | 无风险 |
| rebase | 线性化 | 私有分支、整理历史 | 改变历史，谨慎使用 |

### 实战命令

```bash
# === Merge：保留历史 ===
git checkout main
git merge feature                      # 快进合并（无冲突）
git merge feature --no-ff              # 创建合并 commit（保留分支历史）

# === Rebase：线性化历史 ===
git checkout feature
git rebase main                        # 将 feature 的 commit 变基到 main
# 然后回到 main 合合
git checkout main
git merge feature                      # 此时是快进合并，历史线性

# === 解决冲突 ===
# 冲突时会暂停，编辑冲突文件后：
git add resolved-file.txt
git rebase --continue                  # 继续 rebase
git rebase --abort                     # 放弃 rebase

# === 危险操作：已推送的分支不要 rebase ===
# rebase 改变了 commit hash，会与远程冲突
```

---

## 4.5 远程协作：同步与推送

远程仓库（remote）是团队协作的桥梁。`git push` 上传本地变更，`git pull` 下载远程变更。`git fetch` 只下载不合并，适合先检查再决定。

### Remote 操作对比

| 命令 | 作用 | 本地变更 | 远程变更 |
|------|------|----------|----------|
| push | 上传 | → 远程 | 无 |
| pull | 下载并合并 | 无 | → 本地 |
| fetch | 只下载 | 无 | 下载到本地缓存 |

### 实战命令

```bash
# === 查看远程 ===
git remote                             # 远程仓库名称
git remote -v                          # 名称 + URL
git remote show origin                 # 远程详情

# === 推送 ===
git push origin main                   # 推送主分支
git push -u origin main                # 推送并设置上游（首次）
git push                               # 推送到已设置的上游
git push --force                       # 强制推送（危险，覆盖远程）

# === 拉取 ===
git pull origin main                   # 拉取并合并
git pull                               # 拉取已设置的上游
git pull --rebase                      # 拉取并用 rebase 合并

# === 只下载不合并 ===
git fetch origin                       # 下载远程变更
git fetch                              # 下载所有远程
git diff origin/main                   # 比较本地与远程
git merge origin/main                  # 手动合并
```

---

## 4.6 暂存与回退：后悔药

Git 提供多种"后悔药"：`git stash` 暂存当前修改，`git reset` 回退 commit，`git revert` 创建反向 commit。选择哪种取决于场景和是否已推送。

### 回退方式对比

| 命令 | 作用 | 已推送? | 风险 |
|------|------|---------|------|
| reset | 删除 commit | 未推送 | 改变历史 |
| revert | 创建反向 commit | 已推送 | 不改变历史 |
| stash | 暂存工作区 | — | 无风险 |

### 实战命令

```bash
# === 暂存工作区 ===
git stash                              # 暂存当前修改
git stash list                         # 查看暂存列表
git stash pop                          # 恢复最近暂存
git stash apply stash@{1}              # 恢复指定暂存
git stash drop stash@{0}               # 删除指定暂存

# === 回退 commit（未推送时）===
git reset HEAD~1                       # 回退 1 个 commit，保留修改
git reset --hard HEAD~1                # 回退 1 个 commit，丢弃修改
git reset --soft HEAD~1                # 回退 1 个 commit，保留暂存

# === 反向 commit（已推送时）===
git revert abc123                      # 创建反向 commit
git revert abc123 def456               # 反向多个 commit

# === 撤销暂存 ===
git restore --staged file.txt          # 取消暂存（新版）
git reset HEAD file.txt                # 取消暂存（传统）

# === 撤销工作区修改 ===
git restore file.txt                   # 恢复到暂存区状态
git checkout -- file.txt               # 恢复到暂存区状态（传统）
```

---

## 4.7 完整协作案例

场景：开发新功能，提交 PR，合并到主分支。

```bash
# === 1. 克隆项目 ===
git clone https://github.com/team/project.git
cd project

# === 2. 创建功能分支 ===
git checkout -b feature-user-auth

# === 3. 开发并提交 ===
git add .
git commit -m "feat: 添加用户认证模块"
git push origin feature-user-auth

# === 4. 创建 Pull Request（在 GitHub 网站或用 gh 命令）===
gh pr create --title "添加用户认证" --body "实现登录/注册功能"

# === 5. 更新分支（有新变更时）===
git checkout main
git pull origin main
git checkout feature-user-auth
git merge main                         # 或 git rebase main

# === 6. 合并后清理 ===
git checkout main
git pull origin main
git branch -d feature-user-auth        # 删除本地分支
git push origin --delete feature-user-auth  # 删除远程分支
```

---

## 4.8 Commit Message 规范

规范的 commit message 让历史可读、可追溯。推荐使用 Conventional Commits 格式：类型 + 简短描述。

### Commit 类型对照

| 类型 | 含义 | 示例 |
|------|------|------|
| feat | 新功能 | `feat: 添加用户登录` |
| fix | Bug 修复 | `fix: 修复登录超时` |
| docs | 文档变更 | `docs: 更新 API 文档` |
| style | 格式调整 | `style: 代码缩进统一` |
| refactor | 代码重构 | `refactor: 重构认证逻辑` |
| test | 测试相关 | `test: 添加登录测试` |
| chore | 构建/工具 | `chore: 更新依赖版本` |

### 实战命令

```bash
# === 规范提交 ===
git commit -m "feat: 添加用户认证模块"
git commit -m "fix: 修复密码校验逻辑错误"

# === 多行提交 ===
git commit -m "feat: 添加用户认证模块" -m "- 实现登录功能
- 实现注册功能
- 添加密码校验"

# === 使用 HEREDOC（推荐）===
git commit -m "$(cat <<'EOF'
feat: 添加用户认证模块

- 实现登录功能
- 实现注册功能
- 添加密码校验

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

---

## 4.9 关键认知

三个核心原则：

1. **commit 是快照不是差异**：每次 commit 记录完整状态，不只是变更
2. **已推送的分支不要 rebase/reset**：改变历史会与远程冲突
3. **公共分支用 merge**：保留历史，方便追溯和回退

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 强制推送覆盖他人 | 团队成员代码丢失 | 避免 `--force`，用 `--force-with-lease` |
| merge 冲突未解决 | 代码混乱或丢失 | 仔细检查冲突标记，逐个文件处理 |
| commit message 不规范 | 历史难以理解 | 使用 Conventional Commits |
| 大文件提交 | 仓库体积膨胀 | 用 `.gitignore` 排除，考虑 Git LFS |

---

> **下一章**：掌握了版本控制后，我们将学习如何把环境装进"集装箱"，消灭"环境不一致"问题。