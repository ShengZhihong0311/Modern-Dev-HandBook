# 第二章：触角——终端（CLI）与数据流转

> **核心哲学**：命令行是开发者与操作系统最高效的对话方式

---

## 2.1 指令执行的三种姿势

Python 代码有三种执行方式。脚本模式（`python script.py`）最常见，用于常规开发。快闪模式（`python -c "code"`）执行一行代码，适合快速验证。模块模式（`python -m module`）确保使用当前环境中的模块，避免 PATH 问题。

### 三种模式对比

| 模式 | 命令 | 适用场景 |
|------|------|----------|
| 脚本模式 | `python script.py` | 常规开发 |
| 快闪模式 | `python -c "code"` | 一行命令、快速验证 |
| 模块模式 | `python -m module` | 标准库工具、确保环境一致 |

### 实战命令

```bash
# === 快闪模式：一行代码 ===
python -c "print(2 ** 10)"              # 输出 1024
python -c "import sys; print(sys.version)"
python -c "import requests; print('OK')"  # 验证库是否安装

# === 模块模式：确保环境一致 ===
python -m http.server 8000             # 本地 HTTP 服务器
python -m pytest tests/                # 运行测试
python -m pip list                     # 查看已安装包
python -m pdb script.py                # 调试脚本
```

---

## 2.2 管道符：数据流的接力

管道符 `|` 是 Unix 设计哲学的核心：一个程序的输出，成为另一个程序的输入。这创造了"数据流水线"，每个程序只做一件事，组合起来解决复杂问题。

### 实战命令

```bash
# === grep：搜索过滤 ===
grep "error" log.txt                   # 搜索关键词
grep -r "TODO" ./src/                  # 递归搜索
grep -n "def main" app.py              # 显示行号
grep -i "warning" log.txt              # 忽略大小写
grep -E "def [a-z_]+\(" script.py      # 正则搜索

# === head / tail：查看两端 ===
head config.yaml                       # 前 10 行
head -n 20 config.yaml                 # 前 20 行
tail log.txt                           # 后 10 行
tail -f log.txt                        # 实时监控（常用）
tail -n 100 -f log.txt                 # 后 100 行 + 实时监控

# === 管道组合 ===
grep "error" log.txt | wc -l           # 统计错误数量
grep "error" log.txt | sort | uniq -c | sort -rn | head -5  # 最常见错误
tail -n 100 log.txt | grep "error"     # 最近 100 行中的错误
ps aux | grep "python" | grep -v grep  # 查看进程（排除自身）
```

---

## 2.3 逻辑控制符：短路执行

Shell 支持逻辑运算符实现"条件执行"。`&&` 是 AND，前面成功才执行后面；`||` 是 OR，前面失败才执行后面。这是"短路执行"：条件满足时才继续，避免错误蔓延。

### 逻辑控制符对比

| 运算符 | 逻辑 | 含义 |
|--------|------|------|
| `&&` | AND | 前面成功才执行后面 |
| `||` | OR | 前面失败才执行后面 |
| `;` | 顺序 | 无论成败都执行 |

### 实战命令

```bash
# === &&：连锁执行（前面成功才继续）===
uv sync && uv run python main.py       # 安装依赖后启动
uv run pytest && git commit -m "fix"   # 测试通过才提交
npm run build && npm run deploy        # 构建成功才部署

# === ||：失败补救 ===
mkdir mydir || echo "目录已存在"
curl -O https://mirror1.com/file || curl -O https://mirror2.com/file
uv run pytest || echo "测试失败"

# === 组合使用 ===
uv run pytest && git commit || echo "无法提交"
mkdir project && cd project || exit 1
test -f config.yaml || touch config.yaml  # 文件不存在则创建
```

---

## 2.4 HTTP 状态码：网络诊断的密码

HTTP 状态码是服务器返回的"结果密码"，首位数字表示类别：2xx 成功、4xx 客户端错误、5xx 服务器错误。诊断网络问题时，状态码是第一线索。

### 状态码分类

| 类别 | 范围 | 含义 | 问题归属 |
|------|------|------|----------|
| 2xx | 200-299 | 成功 | 无问题 |
| 4xx | 400-499 | 客户端错误 | 检查请求 |
| 5xx | 500-599 | 服务器错误 | 检查服务 |

### 常见状态码

| 状态码 | 名称 | 常见原因 |
|--------|------|----------|
| 200 | OK | 正常响应 |
| 400 | Bad Request | 参数缺失、格式错误 |
| 401 | Unauthorized | 缺少认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | URL 错误 |
| 429 | Too Many Requests | 触发限速 |
| 500 | Internal Server Error | 代码异常 |
| 502 | Bad Gateway | 上游服务异常 |
| 503 | Service Unavailable | 服务过载 |
| 504 | Gateway Timeout | 上游超时 |

---

## 2.5 curl：网络触角

curl 是命令行的"浏览器"，可以发送任意 HTTP 请求。调试 API 时，先用 curl 模拟请求，排除客户端因素。

### 实战命令

```bash
# === 基础请求 ===
curl https://api.github.com           # GET 请求
curl -I https://api.github.com        # 只看响应头（状态码）
curl -v https://api.github.com        # 完整请求/响应过程

# === API 交互 ===
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "test"}'

curl -H "Authorization: Bearer TOKEN" https://api.example.com/data

# === 健康检查 ===
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/health
# 输出状态码：200（正常）或 503（异常）

curl -w "响应时间: %{time_total}s\n" -s -o /dev/null https://api.example.com

# === 结合逻辑控制符 ===
curl http://localhost:8001/health && echo "服务正常" || echo "服务异常"
```

---

## 2.6 重定向：数据分流

每个程序有三个标准数据流：stdin（输入）、stdout（输出）、stderr（错误）。重定向符号可以改变它们的去向。

### 重定向符号对比

| 符号 | 作用 | 用途 |
|------|------|------|
| `>` | stdout → 文件 | 保存输出（覆盖） |
| `>>` | stdout → 文件 | 保存输出（追加） |
| `2>` | stderr → 文件 | 保存错误 |
| `2>&1` | stderr → stdout | 合并输出和错误 |
| `<` | 文件 → stdin | 从文件读取 |
| `/dev/null` | 丢弃 | 隐默执行 |

### 实战命令

```bash
# === 输出重定向 ===
python script.py > output.txt          # 输出保存到文件
python script.py >> output.txt         # 输出追加到文件
python script.py 2> error.log          # 错误保存到文件
python script.py > all.log 2>&1        # 输出和错误合并保存
python script.py > /dev/null 2>&1      # 隐默执行（丢弃所有）

# === 输入重定向 ===
python process.py < data.txt           # 从文件读取输入

# === 实用组合 ===
curl -s -o /dev/null -w "%{http_code}" http://example.com > status.txt
nohup python server.py > log.txt 2>&1 &  # 后台运行并记录日志
```

---

## 2.7 完整诊断案例

场景：服务响应变慢，需要诊断。

```bash
# 1. 检查服务状态和响应时间
curl -w "响应时间: %{time_total}s 状态: %{http_code}\n" \
  -s -o /dev/null http://localhost:8081/health

# 2. 查看最近日志中的错误
tail -n 100 /var/log/app.log | grep -i "error\|warn"

# 3. 统计各类型错误数量
tail -n 1000 /var/log/app.log | grep "error" | \
  awk '{print $3}' | sort | uniq -c | sort -rn

# 4. 检查进程内存占用
ps aux | grep "python" | awk '{print $6/1024 "MB " $11}'

# 5. 查看端口占用
netstat -tlnp | grep 8081
```

---

## 2.8 关键认知

三个核心原则：

1. **管道优先**：能用管道组合解决的问题，不要写脚本
2. **模块模式保环境**：`python -m` 比 直接命令更可靠
3. **curl 验证网络**：API 问题先用 curl 模拟，排除客户端因素

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 忘记 `-f` 监控 | 看不到新日志 | 用 `tail -f` |
| grep 匹配自身 | ps 结果包含 grep | 用 `grep -v grep` |
| stderr 混入 stdout | 日志混乱 | 用 `2>` 分离 |
| curl 不看状态码 | 不知道请求结果 | 用 `-w "%{http_code}"` |

---

> **下一章**：有了终端作为触角，我们将学习如何像侦探一样诊断程序问题——读懂 Traceback、理解 Exit Code、追踪变量真相。