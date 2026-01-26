
#### `Sequence` vs `list`

- **`list`**: 指具体的列表实现（Mutable）。
    
- **`Sequence`**: 指任何可以按顺序访问的集合（接口），包括列表、元组、字符串等。
    
- **结论**: 在定义 LangGraph 的 State 时，直接写 `list` 即可。因为 LangGraph 专门写的合并函数 `add_messages` 在运行时总是返回一个标准的 `list`。虽然源码中常写 `Sequence` 是为了兼容性，但在应用层没必要。
    

Python

```
from typing import TypedDict, Annotated
from langgraph.graph import add_messages
from langchain_core.messages import BaseMessage

class AgentState(TypedDict):
    question: str
    # 推荐写法：明确列表内容类型
    messages: Annotated[list[BaseMessage], add_messages]
```

#### `Annotated`

- **含义**: 顾名思义，`Annotated` 就是“带有注释的数据类型”。
    
- **结构**: `Annotated[T, x]`。
    
    - `T`: 真实数据类型（如 `int`, `list`）。**IDE 和静态检查器只看这里。**
        
    - `x`: 附加的元数据（Metadata）。**运行时工具（如 Pydantic, LangGraph）看这里来执行特定逻辑。**
        

**Pydantic 示例（数据校验）：**

Python

```
from typing import Annotated
from pydantic import BaseModel, Field

class User(BaseModel):
    name: str
    # 运行时检查：必须是 int，且必须 > 18
    age: Annotated[int, Field(gt=18, description="用户年龄")]
```

**LangGraph 示例（状态合并）：**

Python

```
# 运行时逻辑：使用 add_messages 函数进行 Append/Upsert，而不是直接覆盖
messages: Annotated[list, add_messages]
```


---

#### `@dataclass` 装饰器

Python 3.7+ 内置装饰器，自动为类生成 `__init__`、`__repr__`、`__eq__` 等方法。

**传统写法（繁琐）：**

```python
class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email
    
    def __repr__(self):
        return f"User(id={self.id}, name={self.name}...)"
```

**dataclass 写法（简洁）：**

```python
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str
```

一行装饰器，自动生成所有那些方法！

**常用功能：**

| 功能 | 写法 |
|------|------|
| 默认值 | `title: str = "Untitled"` |
| 可变默认值 | `items: list = field(default_factory=list)` |
| 不可变 | `@dataclass(frozen=True)` |
| 转字典 | `from dataclasses import asdict; asdict(obj)` |

#### `field(default_factory=...)`

为什么需要？直接写 `items: list = []` 会导致**所有实例共享同一个列表**（Python 经典坑）。

```python
# ❌ 危险
class Chunk:
    tags: list = []

# ✅ 正确 - 每个实例创建独立列表
from dataclasses import dataclass, field

@dataclass
class Chunk:
    tags: list = field(default_factory=list)
```

**vs Pydantic：**
- `dataclass`：轻量，适合内部数据结构
- `Pydantic`（`BaseModel`）：自动验证、JSON 序列化，适合 API 输入输出