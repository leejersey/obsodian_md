### **Python 基础学习内容详解：从语法到函数、面向对象，再到 requests、json 与 asyncio 三大实战常用库**

下面把这一阶段拆成四大模块具体展开。每个模块都标注了「要掌握到什么程度」和「配套小练习」，你可以按顺序逐个攻克。对 AI 应用开发而言，重点不是把 Python 学到"精通编译原理"，而是能顺畅地读写代码、调接口、处理数据。

### **一、Python 基础语法**

这是最底层的地基，目标是能读懂、写出结构清晰的脚本。核心知识点包括：

- **变量与数据类型**：整数、浮点、字符串、布尔值，以及三大核心容器——列表（list）、字典（dict）、元组（tuple）、集合（set）。其中 **字典** 在 AI 开发中极其重要，因为 API 的请求和返回几乎都是字典/JSON 结构。
- **运算符与表达式**：算术、比较、逻辑（and/or/not）、成员运算（in）。
- **控制流**：`if / elif / else` 条件判断，`for` 与 `while` 循环，以及 `break`、`continue`。
- **字符串处理**：格式化（推荐用 f-string，如 `f"结果是 {result}"`）、切片、常用方法（`split`、`join`、`strip`、`replace`）。
- **推导式（Comprehension）**：列表推导、字典推导，是 Python 简洁高效的标志，例如 `[x*2 for x in nums if x > 0]`。
- **异常处理**：`try / except / finally`，这在调用外部 API 时是必备技能（网络会失败、返回会出错）。

一个简单的语法示例，展示 f-string、列表推导与条件判断的组合：

```python
scores = [88, 42, 91, 67, 30]
passed = [s for s in scores if s >= 60]
print(f"共有 {len(passed)} 人及格，平均分 {sum(passed) / len(passed):.1f}")
```

**练习建议**：写一个统计文本中单词出现次数的脚本（用字典累加），能同时练到字符串、循环和字典。

### **二、函数（Functions）**

函数是组织代码、复用逻辑的基本单元，AI 应用里每一个"调用模型""处理结果"的动作通常都封装成函数。你需要掌握：

- **定义与调用**：`def` 关键字、参数、返回值。
- **参数机制**：位置参数、关键字参数、默认参数、可变参数 `*args` 与 `**kwargs`（后者在封装 API 调用时特别常用）。
- **作用域**：局部变量与全局变量的区别。
- **匿名函数 lambda**：配合 `map`、`filter`、`sorted` 使用。
- **类型注解（Type Hints）**：如 `def add(a: int, b: int) -> int:`，虽非强制，但强烈建议养成习惯，能让代码更易读、IDE 提示更准。
- **文档字符串（docstring）**：给函数写简短说明。

示例：一个带默认参数和类型注解的函数：

```python
def greet(name: str, greeting: str = "你好") -> str:
    """返回一句问候语"""
    return f"{greeting}，{name}！"

print(greet("小明"))              # 你好，小明！
print(greet("小红", "早上好"))    # 早上好，小红！
```

**练习建议**：把上一节的"单词统计"改写成一个接收文本、返回统计字典的函数，体会封装与复用。

### **三、面向对象编程（OOP）**

当代码规模变大、需要管理"状态"时（比如一个持有 API key、会话历史的聊天客户端），面向对象就派上用场。核心概念：

- **类与对象**：`class` 定义类，实例化生成对象。
- **构造方法 `__init__`** 与实例属性 `self`。
- **实例方法、类方法（`@classmethod`）、静态方法（`@staticmethod`）**。
- **继承与多态**：子类复用并扩展父类。
- **封装**：用下划线约定表示"私有"属性（如 `_api_key`）。
- **特殊方法（魔术方法）**：如 `__str__`、`__repr__`，让对象更好打印和调试。

一个贴近 AI 应用场景的例子——封装一个简单的对话客户端：

```python
class ChatBot:
    def __init__(self, name: str):
        self.name = name
        self.history = []          # 保存对话记录

    def ask(self, message: str) -> str:
        self.history.append({"role": "user", "content": message})
        reply = f"（{self.name} 收到）：{message}"   # 此处真实场景会调用大模型 API
        self.history.append({"role": "assistant", "content": reply})
        return reply

bot = ChatBot("助手")
print(bot.ask("今天天气怎么样？"))
print(f"共记录 {len(bot.history)} 条对话")
```

理解这个例子，你就基本掌握了后续封装 LLM 客户端的思路。

### **四、三大常用库**

前三个模块是通用编程能力，这一节则是 AI 开发中每天都会用到的具体工具。

#### **1. requests —— HTTP 请求库**

几乎所有大模型 API 都通过 HTTP 调用，`requests` 是最主流的同步请求库。要掌握：

- 发起 GET / POST 请求；
- 设置请求头（headers，尤其是携带鉴权 token 的 `Authorization`）；
- 传递 JSON 数据（`json=` 参数）与查询参数（`params=`）；
- 读取响应：`response.status_code`、`response.json()`、`response.text`；
- 处理超时（`timeout`）与异常。

```python
import requests

resp = requests.post(
    "https://api.example.com/v1/chat",
    headers={"Authorization": "Bearer YOUR_KEY"},
    json={"model": "demo", "messages": [{"role": "user", "content": "你好"}]},
    timeout=30,
)
if resp.status_code == 200:
    data = resp.json()
    print(data)
else:
    print(f"请求失败：{resp.status_code}")
```

#### **2. json —— 数据序列化**

大模型的输入输出普遍是 JSON 格式，Python 内置的 `json` 模块负责在"字符串"与"字典/列表"之间转换：

- `json.dumps(obj)`：把 Python 对象转成 JSON 字符串（写文件、发请求）；
- `json.loads(text)`：把 JSON 字符串转回 Python 对象（解析响应）；
- `json.dump` / `json.load`：直接读写文件；
- 常用参数 `ensure_ascii=False`（正确显示中文）、`indent=2`（美化缩进）。

```python
import json

data = {"name": "小明", "skills": ["Python", "AI"]}
text = json.dumps(data, ensure_ascii=False, indent=2)
print(text)

back = json.loads(text)
print(back["skills"][0])   # Python
```

#### **3. asyncio —— 异步编程**

当你的应用需要同时发起大量请求（比如并发调用模型、批量处理文档）时，异步能大幅提升效率。这是本阶段相对进阶的部分，可稍后再深入。要理解：

- **协程**：用 `async def` 定义、`await` 等待；
- **事件循环**：`asyncio.run(main())` 启动；
- **并发执行**：`asyncio.gather()` 同时运行多个任务；
- 配合异步 HTTP 库（如 `aiohttp` 或 `httpx` 的异步模式），因为普通 `requests` 是同步的、不能直接 `await`。

```python
import asyncio

async def fetch(task_id: int) -> str:
    await asyncio.sleep(1)          # 模拟一次网络请求耗时
    return f"任务 {task_id} 完成"

async def main():
    results = await asyncio.gather(*[fetch(i) for i in range(3)])
    print(results)

asyncio.run(main())   # 三个任务并发，总耗时约 1 秒而非 3 秒
```

理解上面这段代码的关键点在于：三个任务几乎同时执行，总耗时接近单个任务，而不是简单叠加——这正是异步在高并发 AI 调用中的价值所在。

### **学习顺序与心态建议**

这四个模块建议**按序推进**：语法 → 函数 → 面向对象是层层递进的通用能力，务必扎实；三大库中，**requests 和 json 应优先熟练掌握**（几乎立刻用得上），而 **asyncio 可以放到你已经能顺畅调用 API 之后再学**，因为它属于性能优化范畴，初期用同步方式完全能跑通流程。

整个阶段大约投入两到三周即可达到"能动手做项目"的水平。不必追求一次性全部记住，边写边查是常态。等你能独立写出一个"用 requests 调接口、用 json 解析结果、用类封装逻辑"的小脚本时，就可以顺利进入下一阶段的大模型基础学习了。

如果你希望，我可以针对其中某一个模块（比如 requests 或 asyncio）再展开更细致的实战教程和练习题。