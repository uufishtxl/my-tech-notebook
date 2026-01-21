---
index: 4
gemini_id: 96759dcb8cd9423c
synced_at: 2026-01-20T11:39:15.933Z
---

## Q

那我想把这个result 作为 information传入这个chain 要做什么数据转换吗？

system_inform = """Based on the order details provided, inform the user of any missing information.

If the food items are missing, include "Please specify the food items you want to order."

If the delivery address is missing, include "Please provide the delivery address."

If the order date is missing, include "Please provide the date and time for the order."




For example, if both the delivery address and order date are missing, the message should be "Your information is incomplete: Please provide your delivery address and order date."

"""




inform_prompt = ChatPromptTemplate.from_messages([

    ("system", system_inform),

    ("human", "{information}")

])




missing_info_chain = inform_prompt | llm | StrOutputParser()

missing_info_chain.invoke({

    "information": "{'food_items': 'Yes', 'delivery_address': 'Yes', 'order_date': 'No'}"

})

## A

是的，你需要做一个非常简单的**数据转换**。

虽然 LangChain 的 PromptTemplate 很聪明，可以直接处理字符串，但为了让 LLM 看得更清楚，且保证格式兼容，最好的做法是将 Pydantic 对象转换为 **字典 (Dictionary)** 或 **JSON 字符串**。

### 推荐做法：转成字典

你的 `result` 是一个 Pydantic 对象。

*   如果你用的是 **Pydantic v2** (现在的标准)，使用 `.model_dump()`。
    
*   如果你用的是 **Pydantic v1** (旧版)，使用 `.dict()`。
    

#### 代码示例

假设你的 `result` 对象里，缺失的字段是 `None`（比如 `order_date=None`）。

```
# 1. 把对象转成字典 (这是最关键的一步)
# Pydantic v2 写法 (推荐):
info_dict = result.model_dump() 
# 如果报错说没有 model_dump，试用旧版写法: result.dict()

print(info_dict) 
# 输出可能长这样: {'item': 'pizza', 'address': 'Fake St', 'order_date': None}

# 2. 传给 Chain
response = missing_info_chain.invoke({
    "information": info_dict  # 直接把字典传进去
})

```

### 为什么这样做？

LangChain 的 PromptTemplate 在接收到字典类型的输入时，会自动把它渲染成字符串格式给 LLM 看。

LLM 会看到类似这样的文本：

```
Based on the order details provided...
...
User: {'item': 'pizza', 'address': 'Fake St', 'order_date': None}

```

LLM 非常聪明，它看到 `None` 或者空字符串，就能理解这个信息是 **"Missing"** 的，从而根据你的 System Prompt 触发 `"Please provide the date..."` 的逻辑。

### 不需要手动转成 "Yes/No" 格式

注意你给出的例子里写的是 `{'food_items': 'Yes', ...}`。 **你不需要在 Python 侧手动写代码去判断 Yes/No。**

直接把原始数据的字典扔给 LLM（包含具体的值和 None），LLM 自己会判断：

*   看到 `'pizza'` -> 知道提供了。
    
*   看到 `None` -> 知道缺失了。
    

这正是使用 LLM 的优势：**把逻辑判断交给 AI，而不是自己写 if/else。**
