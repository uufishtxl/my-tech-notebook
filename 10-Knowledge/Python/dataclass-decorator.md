# Python dataclass 装饰器

#python #dataclass

---

## `@dataclass` 装饰器

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

---

## 常用功能

| 功能 | 写法 |
|------|------|
| 默认值 | `title: str = "Untitled"` |
| 可变默认值 | `items: list = field(default_factory=list)` |
| 不可变 | `@dataclass(frozen=True)` |
| 转字典 | `from dataclasses import asdict; asdict(obj)` |

---

## `field(default_factory=...)`

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

---

## vs Pydantic

- `dataclass`：轻量，适合内部数据结构
- `Pydantic`（`BaseModel`）：自动验证、JSON 序列化，适合 API 输入输出

## 装饰器`frozen`的妙用：去重

```python
@dataclass(frozen=True)
class WordItem:
    text: str
    pos: str # 词性

words = [
    WordItem("book", "noun"), 
    WordItem("book", "verb"), 
    WordItem("book", "noun") # 重复项
]

unique_words = set(words) 
# 结果自动剩下 2 个！非常高效。
```