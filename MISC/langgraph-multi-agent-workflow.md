# LangGraph Multi-Agent Workflow 核心源码解析

## 创建多智能体 `graph` 的标准范式

### 1. 定义 State 数据结构

`AgentState` 是整个图运行期间的全局“共享黑板”。

```python
from typing import Annotated, Literal
from typing_extensions import TypedDict
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    """
    Shared state for the multi-agent graph.
    Fields:
        messages: Conversation history (auto-merged via add_messages reducer)
        next: Routing decision from the router node
        context: Retrieved RAG context (for DocQA)
        sources: Source documents from vector search
    """
    messages: Annotated[list, add_messages]
    next: Literal["doc_qa", "script_editor", "general"]
    context: str
    sources: list[dict]
```

### 2. 创建空白容器

创建一张空的 StateGraph，将定义好的 State 的数据结构传入进去，界定整个框架的“物理定律”。

```python
from langgraph.graph import StateGraph, END
graph = StateGraph(AgentState)
```

### 3. 注册各个节点 (Node)

通过 `graph.add_node("node_name", node_logic)` 将普通的大脑皮层和打手节点接入网络。

```python
# 注册普通执行大脑
graph.add_node("router", router_node)
graph.add_node("doc_qa", doc_qa_node)
graph.add_node("script_editor", script_editor_node)
graph.add_node("general", general_node)

# 注册行动打手
graph.add_node("tools", ToolNode(SCRIPT_TOOLS))
```

> 💡 **核心哲学：军师与打手的分离架构**
> 在定义 `script_editor_node` 时，我们通过 `bind_tools` 的操作，相当于给“军师”阅读了**“英雄兵器谱”**。军师知道了 4 个函数的名字、参数说明（JSON Schema 格式），也就是知道了手握哪些“兵器可以调用”。
>
> 但军师**只有嘴会传令，却没有手能实操**。对于兵器如何发力执行并影响现实（数据库），需要我们通过 `ToolNode` 来为系统**配备真正的打手实体**。
>
> 配合流程：当“军师”发现需要改变现实场景，举起牌子（LLM 回复里带有 `tool_calls` 标志）时，路由器（Edges）就会说：“传令下去，具体的执行工具出列接任务！”。此时，流程走到了 `tools` 节点由其实际运行 Python 代码。任务完成后，ToolNode 会将战果包装成一条特殊的 `ToolMessage`，顺着边（Edge）扔回给 `script_editor` 军师那，让军师复盘并定夺下一步行动战略。

### 4. 设定起跑线 (Entry Point)

指定这个 MultiAgent 开始干活的地方。

```python
graph.set_entry_point("router")
```

### 5. 编织路网：条件路由 (Conditional Edges)

`router` 节点会根据最近一条 `HumanMessage` 意图，更新下一个节点的目标名称（`next` 属性）到 `AgentState` 中。因此，对于 `router` 到下一个节点的流控（Flow Control），需要依赖带判断逻辑的 `Conditional Edge`。

```python
graph.add_conditional_edges(
    "router",
    route_decision,
    # 语法糖：如果未显式提供如下字典映射表，底层引擎也会自动识别，
    # 只要判断函数返回的字符串与下一个目标节点的名字大小写拼写完全一致即可。
    {
        "doc_qa": "doc_qa",
        "script_editor": "script_editor",
        "general": "general",
    }
)
```

### 6. 清理退出路线 (Normal Edges)

`doc_qa` 和 `general` 都是相对单纯的功能节点。思考或检索完毕后，就表示当轮对话任务结束，因此直接将它们牵引到终点线 `END` 即可。

```python
graph.add_edge("doc_qa", END)
graph.add_edge("general", END)
```

### 7. ReAct 循环结界 (Reasoning and Acting Loop)

而 `script_editor` 作为一个“拥有兵器调度权的军师”，处境最为特殊，它需要被卷入一个闭环流程：

**A.** 军师遇到难题，翻查兵器谱，大喊所需工具出列，并在答复中打上 `tool_calls` 的烙印。
**B.** 打手执行完毕将真实结果包装到 `ToolMessage` 发还给军师，让其基于结果复盘制定下一步。
**C.** `should_continue_tools` 边控逻辑发威：军师如果认为还要组合出招，继续 A → B 循环往复；若长舒一口气决定终止，则不再输出 `tool_calls`，表示任务功德圆满。

```python
# 构建军师向打手下达动作的条件流转
graph.add_conditional_edges(
    "script_editor",
    should_continue_tools,
    {
        "tools": "tools",  # 如果判定还有 tool_calls，流向打手节点
        END: END           # 如果判定任务完毕，直接结束释放进程
    }
)

# 确保打手打完后，必须拿着结果回大帐（军师处）复命
graph.add_edge("tools", "script_editor")
```
