# LangGraph (快速了解)

本文整理了多智能体 (Multi-Agent) 协作系统中的核心机制与避坑指南。

## ⚔️ llm.bind_tools(TOOL_LIST)：兵器谱模式
将工具描述绑定到 LLM 上，这是 **“知行合一”** 的基础认知：
- **逻辑本质 (Awareness)**：这并不代表 LLM 能够直接运行 Python。它只是将工具的 JSON Schema（名称、入参、描述）同步给了 LLM，让 LLM 在做决策时知道自己手头有哪些“兵器”可用。
- **产出结果**：LLM 收到 Prompt 后，会返回一个包含 `tool_calls` 列表的消息对象，明确表达“我要调哪把兵器”，但真正的“挥刀”（函数执行）动作是由后传的 `ToolNode` 完成的。

## ⚙️ Reducer 函数：`add_messages`
在 `State` 定义中，通过 `Annotated[list, add_messages]` 实现消息的自动合并。
- **智能识别**：它极为鲁棒，既能接收单条消息 (`AIMessage`)，也能接收消息列表 (`[AIMessage]`)，并自动将其追加到 `State` 的历史中，无需手动写 `extend` 或 `append`。

## 🎟️ 多代理共享 Tool：回城路由 (Return Ticket)
当多个 Agent (如 `script_editor` 和 `reader_editor`) 共用一个公共 `tools` 节点时，执行完后如何“原路返回”？
- **痛点**：`ToolNode` 执行完后，如果没有标记，系统不知道该把结果丢给哪个 Agent 去写总结。
- **方案**：利用 `Router` 已存下的 `state['next']`。在 `tools` 执行完后，通过一个 `route_after_tools` 条件函数，读取这张“车票”并将控制流打回原来的 Agent 节点。

## 🧬 普通 Edge 与 条件 Edge 的区别
- **普通 Edge (`add_edge`)**：适用于单向、100% 确定的执行流。从 A 必须到 B。
- **条件 Edge (`add_conditional_edges`)**：适用于 **动态决策**。它会根据当前 `State` 中的内容（如是否包含 `tool_calls` 或 `next` 的车牌号），决定下一跳是飞向 `tools`、飞回 Agent 还是直接指向 `END`。

---
> 💡 总结：LangGraph 的核心就是“状态管理”与“有向无环图 (DAG)”的碰撞。通过 `State` 存储票据，让多个 Agent 能在同一个复杂的逻辑网里准确地各自安好。
