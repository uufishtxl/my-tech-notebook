---
index: 3
gemini_id: 70898502c8d39c01
synced_at: 2026-01-20T11:39:15.906Z
---

## Q

structured_llm = llm.with_structured_output(OrderCheck, method="function_calling")

result = structured_llm.invoke("I want to order pizza Salami to Fake Street 123, Fake City")




这时候 result 是什么数据类型？

## A

`result` 的数据类型直接就是 **`OrderCheck` 这个类的实例对象**。

这正是 `with_structured_output` 最强大的地方：它帮你省去了“解析 JSON”这一步，直接把 LLM 的回复变成了你定义好的 Python 对象。

### 具体说明

假设你的 `OrderCheck` 是这样定义的（通常是用 Pydantic）：

```
from pydantic import BaseModel, Field

class OrderCheck(BaseModel):
    item: str = Field(description="The name of the item")
    address: str = Field(description="The delivery address")

```

当你运行：

```
result = structured_llm.invoke("I want to order pizza Salami...")

```

此时 `result` **不是**字符串，也**不是**字典，更**不是** `AIMessage`。

它就是：

```
OrderCheck(item='pizza Salami', address='Fake Street 123, Fake City')

```

### 你可以直接这样用

因为它是对象，你可以直接用 **点号 (`.`)** 取值，会有代码提示，非常舒服：

```
# 这里的 result 类型是 <class '__main__.OrderCheck'>
print(result.item)     # 输出: pizza Salami
print(result.address)  # 输出: Fake Street 123, Fake City

# 而不需要像字典那样写 result['item']

```

### 总结

LangChain 在内部做了这两件事：

1.  **取回数据**：拿到 LLM 返回的 JSON 字符串。
    
2.  **自动转换**：帮你运行了类似 `OrderCheck(**json_data)` 的代码，把 JSON 转成了 Pydantic 对象。
