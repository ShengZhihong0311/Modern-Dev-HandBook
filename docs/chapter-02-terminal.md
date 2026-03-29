# 第二章：触角——终端（CLI）与数据流转

> **核心哲学**：命令行是开发者与操作系统最高效的对话方式

---

## 2.1 指令执行的"姿势"

### 原理：三种运行模式

Python 代码有三种执行方式，每种有不同的适用场景：

| 模式 | 命令 | 本质 | 适用场景 |
|------|------|------|----------|
| **脚本模式** | `python script.py` | 执行文件 | 常规开发 |
| **快闪模式** | `python -c "code"` | 执行字符串 | 一行命令、快速验证 |
| **模块模式** | `python -m module` | 执行模块 | 标准库工具、包内脚本 |

---

### 实战：快闪模式 `python -c`

```bash
# 快速计算
python -c "print(2 ** 10)"              # 输出: 1024

# 查看版本信息（比 python --version 更详细）
python -c "import sys; print(sys.version)"

# 一行 JSON 格式化
python -c "import json, sys; print(json.dumps(json.load(sys.stdin), indent=2))"

# 快速测试某个库是否安装
python -c "import requests; print('OK')"  # 成功则输出 OK，失败则报错
```

**典型用途**：
- CI/CD 脚本中的快速检查
- 管道中的数据处理
- 验证依赖是否正确安装

---

### 实战：模块模式 `python -m`

```bash
# 启动本地 HTTP 服务器（当前目录作为根目录）
python -m http.server 8000

# 格式化代码
python -m black .

# 运行测试
python -m pytest tests/

# 查看 pip 信息
python -m pip list

# 调试脚本
python -m pdb script.py
```

**为什么用 `-m` 而非直接命令？**

| 命令 | 问题 |
|------|------|
| `pytest` | 可能找不到命令（未加入 PATH） |
| `python -m pytest` | **确保使用当前 Python 环境中的 pytest** |

**核心原则**：`python -m` 确保执行的是当前虚拟环境中的模块，避免环境混乱。

---

## 2.2 文本处理与搜索

### 原理：管道符 `|` 的本质

管道符是 Unix 设计哲学的核心：**一个程序的输出，成为另一个程序的输入**。

```
程序 A ──► stdout ──► │ ──► stdin ──► 程序 B
```

这创造了一条"数据流水线"，每个程序只做一件事，组合起来解决复杂问题。

---

### 实战：grep 搜索

**基础用法**：

```bash
# 在文件中搜索关键词
grep "error" log.txt

# 在目录中递归搜索
grep -r "TODO" ./src/

# 显示行号
grep -n "def main" app.py

# 忽略大小写
grep -i "warning" log.txt

# 只显示文件名（不显示内容）
grep -l "import" *.py
```

**进阶用法**：

```bash
# 正则表达式搜索
grep -E "def [a-z_]+\(" script.py      # 搜索函数定义

# 统计匹配行数
grep -c "error" log.txt                 # 输出数字

# 显示上下文
grep -C 3 "error" log.txt               # 显示匹配行前后 3 行

# 排除某些文件
grep -r "import" ./src/ --exclude="*.pyc"
```

---

### 实战：head 与 tail

```bash
# 查看文件前 10 行
head config.yaml

# 查看文件前 20 行
head -n 20 config.yaml

# 查看文件最后 10 行
tail log.txt

# 实时监控日志（最常用）
tail -f log.txt                         # 持续显示新增内容

# 查看最后 100 行并实时监控
tail -n 100 -f log.txt
```

**典型场景**：

| 场景 | 命令 |
|------|------|
| 查看配置文件头部 | `head config.yaml` |
| 监控服务日志 | `tail -f /var/log/app.log` |
| 查看最近报错 | `tail -n 50 error.log | grep "Error"` |

---

### 实战：管道组合

```bash
# 统计日志中的错误数量
grep "error" log.txt | wc -l

# 找出最常出现的错误类型
grep "error" log.txt | sort | uniq -c | sort -rn | head -5

# 查看最近 100 行中的错误
tail -n 100 log.txt | grep "error"

# 查看某个进程的 PID
ps aux | grep "python" | grep -v grep

# 按时间排序日志并提取关键信息
cat log.txt | sort -k1 | grep "2024-03"
```

---

## 2.3 逻辑控制符

### 原理：短路执行

Shell 支持逻辑运算符，实现"条件执行"：

| 运算符 | 逻辑 | 含义 |
|--------|------|------|
| `&&` | AND | 前面成功才执行后面 |
| `||` | OR | 前面失败才执行后面 |
| `;` | 顺序 | 无论成功失败都执行 |

**短路原理**：

```
A && B   → A 成功（exit 0）才执行 B
A || B   → A 失败（exit ≠ 0）才执行 B
```

---

### 实战：`&&` 连锁执行

```bash
# 构建成功才部署
npm run build && npm run deploy

# 测试通过才提交
uv run pytest && git commit -m "fix: bug"

# 安装依赖后启动服务
uv sync && uv run python main.py

# 多步骤流水线
git add . && git commit -m "update" && git push
```

**用途**：确保前置条件满足才执行下一步，避免错误蔓延。

---

### 实战：`||` 失败补救

```bash
# 创建目录，如果已存在则忽略
mkdir mydir || echo "目录已存在"

# 尝试主命令，失败则用备用方案
curl -O https://mirror1.com/file.tar.gz || curl -O https://mirror2.com/file.tar.gz

# 测试失败时输出提示
uv run pytest || echo "测试失败，请检查代码"
```

---

### 实战：组合使用

```bash
# 测试通过则提交，失败则提示
uv run pytest && git commit -m "update" || echo "测试失败，无法提交"

# 创建目录成功则进入，失败则报错
mkdir project && cd project || exit 1

# 检查文件是否存在，不存在则创建
test -f config.yaml || touch config.yaml
```

---

## 2.4 网络触角：curl

### 原理：HTTP 请求模拟

curl 是命令行的"浏览器"，可以发送任意 HTTP 请求并查看响应。

**HTTP 基础概念**：

| 概念 | 说明 |
|------|------|
| Method | GET（获取）、POST（提交）、PUT（更新）、DELETE（删除） |
| Header | 请求元数据（Content-Type、Authorization 等） |
| Body | 请求体（POST/PUT 时发送的数据） |
| Status Code | 响应状态码，指示请求结果 |

---

### 原理：常见 HTTP 状态码

状态码分为五类，首位数字表示类别：

| 类别 | 范围 | 含义 |
|------|------|------|
| 1xx | 100-199 | 信息响应（请求正在处理） |
| 2xx | 200-299 | 成功（请求已成功处理） |
| 3xx | 300-399 | 重定向（需要进一步操作） |
| 4xx | 400-499 | 客户端错误（请求有问题） |
| 5xx | 500-599 | 服务器错误（服务器处理失败） |

**最常见状态码详解**：

| 状态码 | 名称 | 含义 | 常见原因 |
|--------|------|------|----------|
| **200** | OK | 请求成功 | 正常响应 |
| **201** | Created | 资源创建成功 | POST 请求创建新资源 |
| **204** | No Content | 成功但无返回内容 | DELETE 请求成功 |
| **301** | Moved Permanently | 永久重定向 | URL 已变更 |
| **302** | Found | 临时重定向 | 临时跳转 |
| **304** | Not Modified | 资源未修改 | 缓存有效，无需重新获取 |
| **400** | Bad Request | 请求格式错误 | 参数缺失、JSON 格式错误 |
| **401** | Unauthorized | 未认证 | 缺少或无效的认证信息 |
| **403** | Forbidden | 禁止访问 | 无权限访问该资源 |
| **404** | Not Found | 资源不存在 | URL 错误、资源已删除 |
| **405** | Method Not Allowed | 方法不允许 | 用 GET 访问只支持 POST 的接口 |
| **408** | Request Timeout | 请求超时 | 客户端发送请求太慢 |
| **409** | Conflict | 冲突 | 资源状态冲突（如重复创建） |
| **429** | Too Many Requests | 请求过多 | 触发限速（Rate Limiting） |
| **500** | Internal Server Error | 服务器内部错误 | 代码异常、配置错误 |
| **502** | Bad Gateway | 网关错误 | 上游服务返回无效响应 |
| **503** | Service Unavailable | 服务不可用 | 服务过载或正在维护 |
| **504** | Gateway Timeout | 网关超时 | 上游服务响应超时 |

**诊断思路**：

| 状态码范围 | 问题归属 | 排查方向 |
|------------|----------|----------|
| 4xx | **客户端问题** | 检查请求参数、URL、认证信息 |
| 5xx | **服务器问题** | 检查服务日志、服务器状态、配置 |

---

### 实战：基础请求

```bash
# GET 请求（默认）
curl https://api.github.com

# 显示响应头（查看状态码）
curl -I https://api.github.com
# 输出: HTTP/2 200 ...

# 显示完整请求/响应过程
curl -v https://api.github.com

# 保存响应到文件
curl -O https://example.com/file.pdf       # 用原文件名保存
curl -o myfile.pdf https://example.com/file.pdf  # 用自定义名称保存
```

---

### 实战：API 交互

```bash
# POST 请求（发送 JSON）
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "test", "email": "test@example.com"}'

# 带认证的请求
curl -H "Authorization: Bearer TOKEN" https://api.example.com/data

# 上传文件
curl -X POST https://api.example.com/upload \
  -F "file=@localfile.txt"

# 发送表单数据
curl -X POST https://example.com/login \
  -d "username=admin&password=secret"
```

---

### 实战：健康检查

```bash
# 检查服务是否在线
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/health
# 输出: 200（成功）或 503（服务不可用）

# 检查 API 响应时间
curl -w "Time: %{time_total}s\n" -s -o /dev/null https://api.example.com

# 检查本地服务
curl http://localhost:8001/health && echo "服务正常" || echo "服务异常"
```

---

## 2.5 重定向：数据分流

### 原理：三个数据流

每个程序有三个标准数据流：

| 流 | 符号 | 说明 |
|----|------|------|
| stdin（标准输入） | `<` | 程序读取的来源 |
| stdout（标准输出） | `>` | 程序正常输出的去向 |
| stderr（标准错误） | `2>` | 程序错误输出的去向 |

---

### 实战：输出重定向

```bash
# 将输出保存到文件（覆盖）
python script.py > output.txt

# 将输出追加到文件
python script.py >> output.txt

# 将错误保存到文件
python script.py 2> error.log

# 同时保存输出和错误
python script.py > output.txt 2> error.log

# 输出和错误保存到同一文件
python script.py > all.log 2>&1

# 丢弃输出（静默执行）
python script.py > /dev/null

# 丢弃所有输出
python script.py > /dev/null 2>&1
```

---

### 实战：输入重定向

```bash
# 从文件读取输入
python process.py < data.txt

# 配合管道
cat data.txt | python process.py
```

---

## 2.6 实战案例：完整诊断流程

**场景**：服务突然响应变慢，需要诊断

```bash
# 1. 检查服务状态
curl -w "响应时间: %{time_total}s\n" -s -o /dev/null http://localhost:8081/health

# 2. 查看最近日志
tail -n 100 /var/log/app.log | grep -i "error\|warn"

# 3. 统计最近一小时各类型错误数量
tail -n 1000 /var/log/app.log | grep "error" | awk '{print $3}' | sort | uniq -c | sort -rn

# 4. 检查进程内存占用
ps aux | grep "python" | awk '{print $6/1024 "MB " $11}'

# 5. 查看端口占用
netstat -tlnp | grep 8081
```

---

## 2.7 关键认知

### 三个核心原则

1. **管道优先**：能用管道组合解决的问题，不要写脚本
2. `-m` 确保环境**：`python -m` 比 直接命令更可靠
3. **curl 验证网络**：API 问题先用 curl 模拟，排除客户端因素

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 忘记 `-f` 监控 | 看不到新日志 | 用 `tail -f` 实时监控 |
| grep 匹配自身 | ps 结果包含 grep 进程 | 用 `grep -v grep` 过滤 |
| stderr 混入 stdout | 日志混乱 | 用 `2>` 分离错误流 |
| curl 不显示状态码 | 不知道请求是否成功 | 用 `-w "%{http_code}"` |

---

> **下一章预告**：有了终端作为触角，我们将学习如何像侦探一样诊断程序问题——读懂 Traceback、理解 Exit Code、追踪变量真相。