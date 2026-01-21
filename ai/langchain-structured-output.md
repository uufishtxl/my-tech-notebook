# LangChain 结构化输出：软限制 vs 硬限制

#langchain #llm #json #structured-output

让 LLM 稳定输出 JSON 是核心难点，有两种流派：

## 1. 软限制 - JsonOutputParser

**原理**：通过 Prompt Engineering 注入格式说明。

```python
from langchain_core.output_parsers import JsonOutputParser

parser = JsonOutputParser(pydantic_object=UICommand)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helper.\n{format_instructions}"),
    ("user", "{text}")
]).partial(format_instructions=parser.get_format_instructions())

chain = prompt | llm | parser
```

### `.partial()` 技巧

提前"焊死"变量，避免每次调用都传：

```python
# ❌ 每次都要传
chain.invoke({"text": "...", "format_instructions": parser.get_format_instructions()})

# ✅ 提前焊死
prompt = raw_prompt.partial(format_instructions=parser.get_format_instructions())
chain.invoke({"text": "..."})  # 清爽！
```

---

## 2. 硬限制 - with_structured_output

**原理**：利用模型原生的 **Function Calling** 能力。

```python
structured_llm = llm.with_structured_output(UICommand)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Extract user intent."),
    ("user", "{text}")
])

chain = prompt | structured_llm
```

---

## 对比

| 特性 | JsonOutputParser (软) | with_structured_output (硬) |
|------|----------------------|---------------------------|
| 原理 | Prompt 提示词 | 底层 API (Function Calling) |
| 稳定性 | 中 | **高** (强制约束) |
| Token 消耗 | 高 (Prompt 变长) | **低** |
| 适用模型 | 所有 LLM | OpenAI, DeepSeek, Claude |
| 推荐场景 | 普通模型 | **生产环境首选** |
