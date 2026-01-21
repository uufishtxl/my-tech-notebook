# DITA Task 结构与约束

#dita #technical-writing #task

Task 类型用于"如何做"的操作说明，有严格的顺序约束。

## 标准顺序

```
prereq (准备) → context (背景) → steps (步骤) → postreq (收尾)
```

> [!WARNING]
> **绝对不能乱序**。这是 DTD 强制约束。

## step 内部结构

```xml
<step>
  <note>提示（可选，在 cmd 前）</note>
  <cmd>点击 Run 按钮</cmd>
  <info>补充说明（在 cmd 后）</info>
  <stepresult>系统显示成功消息</stepresult>
</step>
```

### 顺序约束

```
note/hazardstatement (0+) → cmd → info/stepxmp/substeps (0+) → stepresult (0-1)
```

| 位置 | 允许的标签 | 逻辑 |
|------|-----------|------|
| cmd **前** | note, hazardstatement | 先预警，后操作 |
| cmd **后** | info, stepresult, stepxmp | 操作后才有结果 |

## 核心标签

| 标签 | 说明 |
|------|------|
| `<prereq>` | 前置条件/准备工作 |
| `<context>` | 背景说明 |
| `<steps>` | 步骤容器 |
| `<cmd>` | 用户**动作**（必填，每步一个） |
| `<info>` | 辅助说明 |
| `<stepresult>` | 系统**反馈** |
| `<postreq>` | 收尾工作 |

## 面试要点

1. **安全第一**：Warning 必须放在 `<cmd>` 之前
2. **动作 vs 结果**：`<cmd>` 是用户动作，`<stepresult>` 是系统反馈
3. **排错能力**：能读懂 DTD 约束的报错信息
4. **样式解耦**：不在 Topic 里调格式，用 CSS/Transformation 统一控制
