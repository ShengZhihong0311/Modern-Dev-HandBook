# 第三章：侦探——程序诊断与异常处理

> **核心哲学**：报错不是敌人，而是程序留给你的"求救信"

---

## 3.1 Traceback：案发现场的线索

Traceback（错误追踪）是 Python 程序崩溃时的"遗言"，记录了错误从发生点到调用入口的完整路径。读懂 Traceback，就能快速定位问题根源。

Traceback 的阅读顺序是从下往上：最后一行是错误类型和消息，往上逐层是调用栈。但排查时要从上往下：最上面的行是错误发生的位置，往下是调用链。

### Traceback 结构示例

```
Traceback (most recent call last):        ← 标题：最近的调用在最前
  File "main.py", line 10, in <module>    ← 错误入口：脚本第 10 行
    process_data(data)
  File "utils.py", line 25, in process_data  ← 调用链：utils 第 25 行
    result = parse_json(content)
  File "utils.py", line 40, in parse_json    ← 错误发生点：第 40 行
    return json.loads(content)
TypeError: the JSON object must be str...  ← 错误类型 + 消息
```

### 常见错误类型对比

| 错误类型 | 含义 | 典型原因 |
|----------|------|----------|
| `TypeError` | 类型错误 | 传入了错误类型的参数 |
| `ValueError` | 值错误 | 参数值不符合预期 |
| `KeyError` | 键不存在 | 字典访问了不存在的键 |
| `AttributeError` | 属性不存在 | 对象没有该属性或方法 |
| `ImportError` | 导入失败 | 模块不存在或路径错误 |
| `ModuleNotFoundError` | 模块不存在 | 导入了未安装的包 |
| `IndexError` | 索引越界 | 列表/数组访问超出范围 |
| `FileNotFoundError` | 文件不存在 | 打开了错误的路径 |
| `PermissionError` | 权限不足 | 没有读写权限 |
| `ZeroDivisionError` | 除零错误 | 除数为 0 |

---

## 3.2 Exit Code：程序的退场密码

Exit Code（退出码）是程序结束时返回给操作系统的数字。0 表示成功，非 0 表示失败。在 CI/CD、脚本连锁执行时，Exit Code 决定了后续流程是否继续。

Shell 的 `&&` 和 `||` 就是基于 Exit Code 工作：前一个命令返回 0（成功）才执行下一个。

### 常见 Exit Code

| 退出码 | 含义 | 常见原因 |
|--------|------|----------|
| 0 | 成功 | 程序正常结束 |
| 1 | 通用错误 | 未捕获的异常 |
| 2 | 命令用法错误 | 参数缺失或无效 |
| 126 | 命令无法执行 | 权限不足或非可执行文件 |
| 127 | 命令未找到 | 命令不存在 |
| 128+N | 信号终止 | 被信号 N 杀死 |
| 137 | 内存溢出 (OOM) | 被 SIGKILL (128+9) 杀死 |
| 130 | Ctrl+C 中断 | 被 SIGINT (128+2) 中断 |

### Exit Code 与信号

| 信号 | 编号 | 含义 | 触发方式 |
|------|------|------|----------|
| SIGINT | 2 | 中断 | Ctrl+C |
| SIGKILL | 9 | 强制终止 | `kill -9` |
| SIGTERM | 15 | 正常终止 | `kill`（默认） |

### 实战命令

```bash
# === 查看上一条命令的 Exit Code ===
echo $?                                # 输出退出码

# === 在脚本中检查 Exit Code ===
python script.py
if [ $? -eq 0 ]; then
    echo "成功"
else
    echo "失败，退出码: $?"
fi

# === Python 中设置 Exit Code ===
# sys.exit(0)  → 成功
# sys.exit(1)  → 失败

# === 测试命令是否存在 ===
which python && echo "Python 已安装" || echo "未找到 Python"

# === 调试内存溢出 ===
dmesg | grep -i "killed process"      # 查看被 OOM 杀死的进程
```

---

## 3.3 类型注解：契约与现实的差距

Python 的类型注解（Type Hint）是"契约"，声明了函数期望的参数类型。但它不是强制约束，运行时不会自动检查。如果传入的类型与注解不符，程序可能正常运行，也可能在某个时刻崩溃。

这导致了两种"幻觉"：代码看起来类型正确（注解说它是 str），但实际运行时它是 int。排查时要看实际值，不要被注解误导。

### 类型注解示例

```python
# 声明：期望 name 是 str，返回 str
def greet(name: str) -> str:
    return f"Hello, {name}"

# 实际调用：传入 int，不会报错（注解不强制）
greet(123)  # 输出 "Hello, 123" — 注解与实际不符，但能运行
```

### 类型检查工具对比

| 工具 | 检查时机 | 强制性 | 适用场景 |
|------|----------|--------|----------|
| 运行时 | 程序执行 | 不强制 | 仅崩溃时暴露 |
| mypy | 开发阶段 | 静态检查 | CI/CD 中预防 |
| pyright | 开发阶段 | 静态检查 | VS Code 内置 |

### 实战命令

```bash
# === 静态类型检查 ===
mypy script.py                         # 检查类型注解一致性
mypy --strict script.py                # 严格模式

# === 运行时强制检查（需库支持）===
# pip install typeguard
from typeguard import typechecked

@typechecked
def greet(name: str) -> str:
    return f"Hello, {name}"

greet(123)  # 报错：TypeError
```

---

## 3.4 变量追踪：真相在内存里

排查变量问题时，不要相信代码"看起来是什么"，要看运行时"实际是什么"。常见陷阱：变量被意外修改、变量名拼写错误、变量作用域混乱。

### 实战调试技巧

```python
# === 快速打印变量 ===
print(f"变量类型: {type(var)}, 值: {var}")

# === 打印调用栈 ===
import traceback
traceback.print_stack()

# === 使用调试器 ===
import pdb; pdb.set_trace()            # 在此处暂停
# 或：python -m pdb script.py          # 从头调试

# pdb 常用命令：
# n — 下一步
# c — 继续运行
# p var — 打印变量
# l — 显示当前位置
# q — 退出
```

### 实战命令

```bash
# === 交互式调试 ===
python -m pdb script.py                # 从头开始调试

# === 断点调试（在代码中插入）===
# import pdb; pdb.set_trace()

# === 查看变量在内存中的地址 ===
python -c "x = [1, 2]; y = x; print(id(x), id(y), id(x) == id(y))"
# 输出地址，判断是否同一对象
```

---

## 3.5 异常处理：捕获与恢复

异常处理不是隐藏错误，而是优雅地处理预期中的失败情况。原则：只捕获你能处理的异常，不要用空的 `except:` 吞掉所有错误。

### 异常处理模式对比

| 模式 | 代码 | 问题 |
|------|------|------|
| 吞掉所有 | `except:` | 隐藏错误，难以排查 |
| 吞掉 Exception | `except Exception:` | 仍太宽泛 |
| 指定类型 | `except ValueError:` | **推荐** |
| 多类型 | `except (ValueError, KeyError):` | 处理多种预期错误 |

### 实战代码

```python
# === 推荐模式：捕获特定异常 ===
try:
    data = json.loads(content)
except json.JSONDecodeError as e:
    print(f"JSON 解析失败: {e}")
    data = {}

# === 记录异常信息 ===
import logging
try:
    process()
except Exception as e:
    logging.exception("处理失败")  # 自动记录完整 Traceback
    raise  # 重新抛出，不隐藏

# === 多异常处理 ===
try:
    value = data[key]
except KeyError:
    value = None
except TypeError:
    value = default
```

---

## 3.6 完整诊断案例

场景：程序崩溃，输出 Traceback，需要定位根因。

```
Traceback (most recent call last):
  File "app.py", line 50, in run
    result = handler.process(request)
  File "handler.py", line 20, in process
    data = parse_input(request.body)
  File "parser.py", line 35, in parse_input
    return json.loads(body)
TypeError: the JSON object must be str, bytes or bytearray, not NoneType
```

诊断步骤：

```bash
# 1. 定位错误发生点：parser.py 第 35 行
# 2. 错误类型：TypeError，传入 None 而非 str
# 3. 追溯调用链：request.body 是 None

# === 检查相关代码 ===
cat parser.py | head -n 40 | tail -n 10

# === 添加调试信息 ===
# 在 parser.py 第 35 行前添加：
# print(f"body 类型: {type(body)}, 值: {body}")

# === 重新运行 ===
python app.py

# === 如果需要更深入调试 ===
python -m pdb app.py
# 在 parser.py:35 设置断点
```

---

## 3.7 关键认知

三个核心原则：

1. **Traceback 从下读**：先看错误类型，再追溯调用链
2. **Exit Code 0 才成功**：非零码意味着失败，`$?` 查看上一条命令结果
3. **相信运行时**：类型注解是"契约"而非"事实"，调试时看实际值

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| 空 except 吞错误 | 程序"正常"但结果异常 | 捕获特定异常类型 |
| 被注解误导 | 代码类型"正确"但崩溃 | 检查实际运行值 |
| 忽略 Exit Code | 脚本连锁执行时错误蔓延 | 用 `&&` 确保前置成功 |
| 变量被意外修改 | 值不符合预期 | 打印 type 和 id 定位 |

---

> **下一章**：掌握了诊断能力后，我们将学习如何用 Git 管理代码的演进史，而不是单纯的备份。