---
index: 2
gemini_id: de37f8354068cb69
synced_at: 2026-01-20T11:39:15.891Z
---

## Q

这段怎么这么奇怪，没有让用户提供today，但是又在花括号里用了 {today}




system_time = """Identify and rewrite the time to match the correct format.

If the provided time is not in the format '%Y-%m-%d %H:%M', rewrite the complete question, keep everything unchanged, despite the time"




Today is: {today}




Important: The correct format, take a look at the example:

Example:

User: 'I want to order a pizza Salami to the Fakestreet 123 for 9:00'

Desired: 'I want to order a pizza Salami to the Fakestreet 123 for 2024-05-30 09:00'

"""




prosystem_time_prompt = ChatPromptTemplate.from_messages(

    [

        ("system", system_time),

        ("human", "{question}"),

    ]

)

today是python关键字？

## A

并不是。**`today` 绝对不是 Python 的关键字**，也不是 Python 自动生成的变量。

之所以你会觉得奇怪，是因为你看到的这段代码只是\*\*“定义模版”**（Template Definition）的阶段，而**“填空”\*\*（Injection/Formatting）的步骤在后面，还没发生。

这里发生了两件事，导致了你的困惑：

### 1\. `{today}` 是个“坑”（Placeholder）

在 LangChain（以及 Python 的 f-string 或 `.format()`）里，花括号 `{variable_name}` 的意思是：**“这里给我留个位置，名字叫 `today`，等会儿真正运行的时候，我会传一个值给你塞进去。”**

它就像合同里的 `____(签字日期)`，并不是说“签字日期”是法律术语，而是留给最后那个人填的。

### 2\. 真正的“填空”动作在后续代码里

你觉得奇怪是因为你没看到后续调用的代码。在使用这个 `prosystem_time_prompt` 时，必须显式地把 `today` 的值传进去，否则程序会报错（KeyError）。

后续运行的代码大概率长这样：

```
from datetime import datetime

# 1. 获取当前真实时间
current_date = datetime.now().strftime("%Y-%m-%d")

# 2. 调用链的时候，把 today 和 question 一起传进去
# 注意看这里！这才是 {today} 被赋值的地方
response = chain.invoke({
    "question": "I want a pizza for 9:00",
    "today": current_date  # <--- 这里填坑
})

```

* * *

### 为什么这么写？（为什么要留个坑？）

为了**动态性**。

如果不写 `{today}` 而是在 prompt 里写死 `Today is: 2025-12-10`，那明天这个 Prompt 就失效了。 通过写成 `{today}`，开发者告诉 LangChain：“每次运行这个 Prompt 时，不管是谁在调用，都必须给我传入一个叫 `today` 的参数，告诉我今天是几号。”

### 另一种可能：使用了 `partial`

有些高级写法会在定义完 template 后，马上用 `partial` 把 `today` 绑定死，这样以后调用就不用每次都传了。这可能也是你没看到“用户提供 today”的原因：

```
from datetime import datetime

# 获取当前时间
now = datetime.now().strftime("%Y-%m-%d")

# 提前把 {today} 这个坑填上
final_prompt = prosystem_time_prompt.partial(today=now)

# 这样以后调用只需要传 question，不用管 today 了
final_prompt.invoke({"question": "..."}) 

``` 

**总结：** `today` 只是程序员起的一个变量名（就像 `question` 一样），它是 LangChain 模版语法的一部分，用来占位的，等待后续代码通过字典传入具体的值。
