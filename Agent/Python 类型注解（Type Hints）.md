### **Python 类型注解（Type Hints）详解：类型注解是给变量、参数和返回值"标注它应该是什么类型"的语法——它不强制、不影响运行，却能大幅提升代码可读性、让 IDE 精准提示、让工具提前发现 bug，是编写专业级 AI 应用代码的重要习惯**

上一节讲 lambda 时，我们的函数签名还是"裸"的——看一个函数，你无法一眼判断它该传什么、返回什么。类型注解正是来解决这个问题的。它是现代 Python（尤其是 3.5 以后）中越来越重要的特性，几乎所有主流的 AI 库、Web 框架（如你后面会学的 FastAPI）都大量使用类型注解。理解它，不仅能让你自己的代码更清晰，也能让你读懂那些专业库的源码和文档。本节会讲清它的语法、常见类型的写法、一个关键的认知误区，以及它在 AI 开发中的实际价值。

### **一、什么是类型注解：一份"类型说明书"**

类型注解就是在变量、参数、返回值旁边，用特定语法标注"它预期是什么类型"。先看一个对比。没有注解的函数：

```python
def greet(name, times):
    return name * times
```

光看这个定义，你很难确定 `name` 和 `times` 该传什么。加上类型注解后，意图一目了然：

```python
def greet(name: str, times: int) -> str:
    return name * times
```

现在任何人都能立刻明白：`name` 应该是字符串、`times` 应该是整数、函数会返回一个字符串。类型注解本质上是一份**写在代码里的、机器和人都能读的"说明书"**。

### **二、最关键的认知：注解不强制、不检查**

这是理解类型注解**最重要、也最容易误解**的一点：**Python 的类型注解只是"提示"，它在运行时完全不会被强制执行，也不会自动检查。** 换句话说，即使你违反了注解，程序照样运行：

```python
def add(a: int, b: int) -> int:
    return a + b

print(add(3, 5))         # 8，正常
print(add("你好", "世界"))  # 你好世界！照样运行，Python 不报错
```

上面第二次调用明明传了字符串（违反了 `int` 注解），但 Python **不会报错**，因为注解不参与运行时的类型检查。这一点和 Java、C++ 等"静态类型语言"有本质区别。

那注解到底有什么用？它的价值体现在三个"运行之外"的地方：

- **给人看**：让阅读代码的人（包括未来的你）快速理解函数的用法。
- **给 IDE 看**：编辑器（如 VS Code、PyCharm）能据此提供精准的自动补全、参数提示和错误高亮。
- **给工具看**：静态检查工具（如 `mypy`、`pyright`）能在你**运行代码之前**就扫描出类型不匹配的潜在 bug。

理解了"注解是给人和工具看的提示，而非运行时的强制约束"，你就抓住了类型注解的精髓。

### **三、基础类型的注解写法**

最常见的注解就是四个基本类型：

```python
name: str = "小明"          # 字符串
age: int = 25              # 整数
height: float = 1.75       # 浮点数
is_student: bool = True    # 布尔
```

变量注解的语法是 `变量名: 类型 = 值`。而在函数中，参数用 `参数名: 类型`，返回值用 `-> 类型` 写在括号后、冒号前：

```python
def calculate_bmi(weight: float, height: float) -> float:
    """计算 BMI 指数"""
    return weight / (height ** 2)
```

如果一个函数**没有返回值**（即返回 `None`），惯例是标注 `-> None`：

```python
def log_message(msg: str) -> None:
    print(f"[日志] {msg}")     # 只做动作，不返回值
```

### **四、容器类型的注解**

对列表、字典等容器，不仅能标注"它是个列表"，还能标注"里面装的是什么类型"，这让注解更精确。

在**较新的 Python（3.9+）** 中，可以直接用内置类型的小写形式：

```python
names: list[str] = ["小明", "小红"]              # 元素是字符串的列表
scores: dict[str, int] = {"小明": 90}            # 键是str、值是int的字典
point: tuple[int, int] = (3, 5)                 # 两个整数的元组
tags: set[str] = {"ai", "python"}               # 字符串集合
```

在函数中使用：

```python
def average(scores: list[float]) -> float:
    return sum(scores) / len(scores)

def count_words(text: str) -> dict[str, int]:
    result: dict[str, int] = {}
    for word in text.split():
        result[word] = result.get(word, 0) + 1
    return result
```

> 补充：在更老的版本（3.8 及以前）里，需要从 `typing` 模块导入大写形式，如 `from typing import List, Dict`，然后写成 `List[str]`、`Dict[str, int]`。新代码优先用小写内置形式即可，了解旧写法是为了读懂老代码。

### **五、typing 模块：更丰富的注解工具**

对于更复杂的场景，`typing` 模块提供了一系列特殊工具。下面几个是 AI 开发中最常用的。

#### **1. Optional：可能是某类型，也可能是 None**

当一个值"要么是某类型，要么是 `None`"时（这在处理可能失败的操作、可选参数时极常见），用 `Optional`：

```python
from typing import Optional

def find_user(user_id: int) -> Optional[str]:
    """找到返回用户名，找不到返回 None"""
    users = {1: "小明", 2: "小红"}
    return users.get(user_id)    # 可能返回 str，也可能返回 None

# Optional[str] 等价于 str | None
```

`Optional[X]` 完全等价于 `X | None`。它明确告诉调用者："这个结果可能为空，你得处理 `None` 的情况"，能有效提醒你避免 `NoneType` 相关的错误。

#### **2. Union 与新式的 | 语法：多种可能的类型**

当一个值可能是几种类型之一时，用 `Union`；在 **Python 3.10+** 中更推荐用简洁的 `|` 竖线语法：

```python
from typing import Union

def to_number(value: str) -> Union[int, float]:   # 旧写法
    ...

def to_number(value: str) -> int | float:          # 新写法（3.10+），更简洁
    if "." in value:
        return float(value)
    return int(value)
```

#### **3. Any：放弃类型检查的"万能牌"**

`Any` 表示"任何类型都可以"，相当于关闭了对这个值的类型检查。当你确实无法确定类型，或不想约束时使用——但应尽量少用，因为用得越多，注解带来的好处就越少：

```python
from typing import Any

def process(data: Any) -> Any:      # 什么都能接、什么都能返回
    return data
```

#### **4. Callable：标注"参数是一个函数"**

呼应上一节的 lambda 和高阶函数——当参数本身是个函数时，用 `Callable` 标注它的输入输出：

```python
from typing import Callable

def apply(func: Callable[[int], int], value: int) -> int:
    """func 是一个：接收一个 int、返回一个 int 的函数"""
    return func(value)

print(apply(lambda x: x * 2, 5))    # 10
```

`Callable[[参数类型列表], 返回类型]`——前面的方括号列出参数类型，后面是返回类型。

### **六、类型别名：给复杂类型起个名字**

当一个类型写起来很长、又反复出现时（这在 AI 开发中很常见，比如消息列表的结构），可以定义**类型别名**让代码更清爽：

```python
# 定义别名：一条消息是 str 到 str 的字典
Message = dict[str, str]
# 一段对话是消息的列表
Conversation = list[Message]

def add_message(history: Conversation, role: str, content: str) -> Conversation:
    history.append({"role": role, "content": content})
    return history

messages: Conversation = []
messages = add_message(messages, "user", "你好")
```

用 `Message` 和 `Conversation` 这样的名字，比每次都写 `list[dict[str, str]]` 清晰得多，也更能表达业务含义。

### **七、AI 开发实战：带完整类型注解的函数**

把本节知识综合起来，看一个贴近真实场景的例子——一个封装了大模型调用的函数，配上完整的类型注解。你会发现，加了注解后，这个函数的"契约"变得极其清晰：

```python
from typing import Optional

# 类型别名，让签名更易读
Message = dict[str, str]

def chat(
    prompt: str,
    history: Optional[list[Message]] = None,
    model: str = "demo-v1",
    temperature: float = 0.7,
) -> dict[str, str | bool]:
    """调用大模型，返回包含成功标志和回复的字典"""
    if history is None:              # 呼应上一节讲的可变默认参数陷阱
        history = []

    history.append({"role": "user", "content": prompt})
    # ... 真实场景：调用 API ...
    reply = f"（模拟回复）：{prompt}"

    return {"success": True, "reply": reply}

# 调用时，IDE 会根据注解提供精准提示
result = chat("你好", temperature=0.9)
print(result["reply"])
```

这个函数把本节几乎所有要点都用上了：基础类型注解（`str`、`float`）、容器注解（`list[Message]`）、`Optional`、类型别名、`-> None` 之外的返回类型标注。任何人拿到这个函数，不用读实现，光看签名就知道该怎么用、会得到什么——**这正是专业 AI 项目代码的标准样貌**，也是你后面学 FastAPI 时会大量见到的写法（FastAPI 甚至直接利用类型注解来做数据校验）。

### **八、几条实用建议**

在结束本节前，给出一些落地建议：

- **从函数签名开始注解**。哪怕不给每个局部变量都加注解，也应该给函数的**参数和返回值**加上——这是投入产出比最高的部分。
- **不必追求 100% 覆盖**。类型注解是"渐进式"的，你可以只给关键、复杂的地方加注解，简单明显的地方留白也无妨。
- **想要真正的检查，请用工具**。如果希望注解能帮你抓 bug，安装并运行 `mypy` 或在编辑器里启用 `pyright`，它们会在运行前就报告类型不匹配。
- **注解要保持诚实**。如果注解写的是 `int` 但实际经常传别的，那注解就成了误导。让注解和真实行为一致，它才有价值。

### **小结与练习建议**

本节的核心记忆点有三个：**类型注解的语法是"参数名: 类型"和"-> 返回类型"，用于标注预期类型**；**注解在运行时不强制、不检查，它的价值在于给人、给 IDE、给静态检查工具（如 mypy）看**；以及 **`Optional[X]`（即 `X | None`）表示可能为空、`Union`/`|` 表示多种类型、`Callable` 标注函数参数**这几个高频工具的用法。

建议动手做一个综合练习来巩固：**给你之前写过的几个函数补上完整的类型注解——比如一个 `safe_divide(a: str, b: str) -> Optional[float]` 的安全除法（失败返回 None）、一个 `rank_results(results: list[dict[str, float]]) -> list[dict[str, float]]` 的排序函数；再定义一个类型别名来简化其中重复的复杂类型。如果条件允许，安装 `mypy` 跑一遍，看它能否发现你故意制造的类型不匹配。** 这个练习能把注解语法和实际函数结合起来，加深理解。

掌握了类型注解，你的函数签名会变得专业而清晰，也为后面学习 FastAPI、Pydantic 等大量依赖类型的工具打下基础。接下来可以顺利进入函数模块的最后一部分——"文档字符串（docstring）"的学习。如果你希望，我可以针对 `mypy` 的实际使用，或泛型（`TypeVar`、`Generic`）等进阶类型注解再展开深入讲解。