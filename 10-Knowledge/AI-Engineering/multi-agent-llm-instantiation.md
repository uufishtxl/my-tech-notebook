# Multi-Agent Design: Stateless LLM Instantiation

在构建类似 LangGraph 的多智能体系统（Multi-Agent System, MAS）时，推荐采用**按需实例化（On-Demand / Stateless Instantiation）**的设计模式。

## 核心模式

不要在全局只初始化一个 `llm` 对象，而是编写一个工厂函数来根据上下文动态创建 LLM 实例。

```python
def _get_llm(feature="default", **kwargs):
    config = settings.LLM_CONFIG.get(feature, {})
    defaults = {"temperature": 0.3}
    defaults.update(kwargs)
    
    return ChatOpenAI(**defaults)

def router_node(state):
    # 根据任务需求，在此处实例化一个特定的 LLM
    llm = _get_llm(feature="router", temperature=0)
    # ...
```

## 这种设计的好处

### 1. 差异化配置 (Granular Configuration)
这是多智能体系统的核心优势之一。我们可以为不同职责的 Agent 分配完全不同的参数：

| Agent | Temperature | 理由 |
| :--- | :--- | :--- |
| **Router** | `0.0` | **完全确定性**。必须 100% 准确分类（如 JSON/XML 输出），不需要任何创造性。 |
| **Tool Calling** | `0.0` | **严格指令执行**。调用 API 或 SQL 查询，参数必须精准无误，禁止幻觉。 |
| **RAG / QA** | `0.3` | **严谨且自然**。基于事实回答，但也需要组织流畅的语言。 |
| **Creative / General** | `0.7` | **发散性思维**。闲聊、创意写作、头脑风暴，需要多样性。 |

### 2. 状态隔离 (State Isolation)
虽然大多数 LLM 客户端本身是无状态的（Stateless），但显式地每次创建一个新对象可以：
- **避免隐式上下文污染**：确保不会因为复用了某个带有 history 的对象而泄露信息。
- **线程安全**：在并发环境中，每个线程拥有独立的可以修改配置的对象。

### 3. 模型混用 (Model Mixing)
允许为简单的任务（如 Router）使用**小模型**（如 `gemini-flash`），为复杂任务（如 Creative）使用**大模型**（如 `gpt-4`），从而优化**成本和响应速度**。
