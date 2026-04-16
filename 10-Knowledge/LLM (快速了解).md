## LLM Temperature 参数控制
- **1 (灵活/发散)**：生成具有创造性、多样性的文本。适合写作、头脑风暴。
- **0 (收紧/确定)**：每次输出几乎完全一致。适合信息抽取、结构化 JSON 输出、代码生成等严谨场景。

## Langchain 结构化输出坑点
在使用 Langchain 的 `.with_structured_output()` 等特性时：
- **兼容性问题**：并非所有模型都完美支持 OpenAI 格式的 API。例如 Deepseek 等部分模型可能在原生 API 层面不支持 `response_format={ "type": "json_object" }` 字段。
- **应对方案**：在 Prompt 中强烈要求输出 JSON，并辅以 Langchain 的标准输出解析器（Output Parsers）做兜底拦截。
## LCEL 的输入序列化
把 `json.dumps` 挂在管道 `|` 上（例如通过 `RunnablePassthrough.assign`）。这不是多此一举，而是为了**关注点分离（解耦）**，确保喂给大模型的永远是标准无歧义的 JSON 字符串，防止 Python“方言”（比如单引号）引发大模型幻觉。
## ⛓️ LCEL 管道输入预处理
通过 `RunnablePassthrough.assign` 或 `lambda` 映射完成**序列化**：
- **核心目的**：关注点分离（解耦）。确保入口只接收纯净的 Python 对象（List/Dict），而复杂的 JSON 字符串转换逻辑被封装在管道内部。
- **防止幻觉**：避免直接向 Prompt 注入 Python 对象的 `__str__` 表达形式（如单引号），统一使用 `json.dumps` 确保 LLM 接收的是标准的、无歧义的 JSON。
## 🛠️ LCEL 数据管道的“外科手术” (Local vs. Full)
在数据送往 Prompt 之前，根据输入数据的形态（List 或 Dict），我们有两套不同的“手术方案”：
### 1. **全量包装 (Dict Wrapper)**
- **场景**：输入是**单变量**或**非字典对象**（如一个 List `['word1', 'word2']`）。
- **方法**：直接定义 `{"key": lambda ...}`。
- **效果**：它会把之前的所有数据丢弃，重新打包成一个全新的字典送给 Prompt。
### 2. **局部拦截 (RunnablePassthrough.assign)** 🌟
- **场景**：输入已经是**多参数字典**。
- **方法**：使用 `.assign(column=lambda x: ...)`。
- **效果**：
    - **透传 (Pass-through)**：原有的所有 key（如 `target_word`）会自动流向下一关，不需要你手动重复赋值。
    - **拦截 (Intercept)**：你只针对那个需要“特殊照顾”的字段进行序列化处理。
- **金句总结**：**“我不关心的数据（String）让它飞，我不放心的数据（List）我来截。”**
## LLM 返回的数据类型流转

## 🧠 LLM 注意力与历史裁剪 (Attention Strategy)
在工程实践中，如何排布消息顺序直接决定了 LLM 的“智商”：
- **System Prompt 置顶 (Top-loading)**：将最重要的指令（如 RAG 检索到的 Context）放在消息队列的最前面。LLM 对序列两端的注意力最集中，置顶能有效防止 LLM 在长对话中由于 `Lost in the Middle` 而产生幻觉。
- **历史会话裁剪 (Message Trimming)**：
    - 一般建议保持 **3-10 轮** 历史。
    - **权衡**：不传历史会导致无法进行“指代消解”（如：用户问“那这个呢？”）；传得太长会稀释系统指令的权重并虚耗 Token 成本。

清晰界定大模型结构化输出在系统中的三种物理形态转换：
1. **纯文本 (String)**：网络传回的原始薛定谔 JSON。
2. **Pydantic 实例 (Object)**：经 `with_structured_output` 校验后，获得强类型和默认值保护的“防弹衣”形态。
3. **Python 字典 (Dict)**：调用 `.model_dump()` 剥离类属性，将其降维成 Web 框架可直接解析的朴素形态。

- **场景**：处理大模型输出的完整生命周期，最终将安全、合规的数据封装成 HTTP Response 返回给前端或下游系统。