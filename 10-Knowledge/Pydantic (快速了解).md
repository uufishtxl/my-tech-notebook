## Pydantic 中 model_dump() 的作用
用于将 Pydantic 验证好的结构化对象（Model 实例）转换回标准的 Python **字典 (Dictionary)**。
- 场景：通常在封装 Response 返回给前端，或者将数据传给不支持 Pydantic 对象的第三方库时使用。
## Pydantic 约束字符串可选范围
### 方法一：使用 `Literal` (极简派)

`Literal` 是 Python `typing` 模块自带的类型提示。
```Python
from typing import Literal
from pydantic import BaseModel, ValidationError

class ScenarioItem(BaseModel):
    # 直接在类型注解里写死允许的字符串
    tag: Literal["Office", "Hospital", "Home"]
```

**特点：**
- **优点**：极其简单、轻量。不需要额外定义任何类，随用随写。
- **缺点**：如果你的标签有 20 个，这行代码会变得非常长且难看；而且这个列表无法被其他函数轻易复用（比如你想用 `for` 循环把所有支持的标签打印出来，`Literal` 做不到）。

---

### 方法二：使用 `Enum` (结构派)
`Enum` 是 Python 自带的枚举类。你需要先单独定义一个“词典”，然后再把它作为类型传给 Pydantic。
**代码演示：**

```Python
from enum import Enum
from pydantic import BaseModel, ValidationError

# 1. 先定义约束范围 (注意这里继承了 str，这是为了让 Pydantic 直接把它当字符串处理)
class ContextTag(str, Enum):
    OFFICE = "Office"
    HOSPITAL = "Hospital"
    HOME = "Home"

# 2. 在模型中使用
class ScenarioItem(BaseModel):
    tag: ContextTag
```

**特点：**
- **优点**：
    1. **可遍历**：你可以随时 `list(ContextTag)` 获取所有标签，方便用来生成给大模型的 Prompt。
    2. **高复用**：任何需要用到这个标签集的地方，直接 `import ContextTag` 即可。
    3. **防拼写错误**：在代码逻辑里写 `if tag == ContextTag.OFFICE:`，IDE 会有自动补全，不怕把 "Office" 拼成 "Ofice"。
- **缺点**：比 `Literal` 多写了几行代码。