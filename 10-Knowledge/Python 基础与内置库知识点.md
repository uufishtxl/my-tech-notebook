## Timezone 知识点
处理时间时，务必带上时区信息（Timezone-aware），避免跨时区业务出错。
- **获取当前时区时间**: (Django 环境下首选)
```Python
from django.utils import timezone
now = timezone.now()
```
- **获取当前本地时间**: 
```Python
from datetime import datetime
now = datetime.now()
```
- **提取日期元素**: 拿到时间对象后，可以直接访问属性：
```Python
year = now.year
month = now.month
day = now.day
weekday = now.weekday() # 返回 0-6，0代表周一
```

## 列表与字典推导式 (Comprehensions)
用于高效、优雅地过滤和转换数据。配合 `if` 语句可以实现“边遍历边过滤”。
- **列表推导式**: `[x for x in items if x > 0]` (提炼大于0的元素)
- **字典推导式**: `{k: v for k, v in my_dict.items() if v is not None}` (剔除字典中 value 为空的键值对)

## 动态枚举
利用 `Enum("ClassName", dict, type=str)` 函数式 API，在运行时根据外部列表批量生成枚举类。核心技巧在于 `type=str` 混入（Mixin），使得生成的枚举成员天然就是字符串，无缝兼容 JSON 序列化。

- **场景**：标签（如 `ContextTag`）允许的列表需要从外部维护，但又必须同时作为**规则**注入到 LLM Prompt 中，并作为**校验器**用于 Pydantic 解析时。
## 🐍 Python 的单行代码换行编写技巧

## 🐍 Python 列表的“倒序”与“负数切片”
除了常用的 `reverse()` (原地反转) 和 `reversed()` (迭代器)，Python 的切片语法在处理“最近消息”时极其强大：
- **负数索引切片**：`list[-N:]` 
    - **含义**：从倒数第 N 个元素开始，一直截取到最后。
    - **妙用**：在 LLM 聊天机器人中用于 **AGENT_HISTORY_WINDOW**（历史滑窗）裁剪。
    - **鲁棒性**：如果列表长度小于 N，Python 不会报错，而是会智能地“有多少取多少”。

当你写一段很长的代码（比如：长字典定义、列表推导、或者是 **LangChain 的 | 管道**）时，为了不让代码横向拉出“银河系”，你可以利用**“隐式续行”**：
### 1. 利用圆括号 
`()` 的魔法 🌟
- **做法**：将长代码整体包裹在 `( ... )` 中。
- **收益**：
    - **无需反斜杠**：不再需要难看的 `\` 续行符。
    - **自由对齐**：可以根据逻辑在每个管道符 `|` 前换行，极度整洁。
    - **示例**：
```Python
chain = (
	RunnablePassthrough.assign(...)
	| prompt
	| llm
)
```

### 2. 为什么不推荐反斜杠 `\` ？

- **脆弱性**：如果在 `\` 后面不小心多了一个空格，Python 会报错 `SyntaxError`。
- **可读性**：容易导致排版错位，让代码显得杂乱无章。