
## 基础概念

* DITA Topic: 类似于 Component
* DITA Map: 类似于 Router
* Content Reuse: 类似于 Python Import / Variables
	* Topic 级复用 ≈ import module，比如《如何登录》，在《管理员手册》中会引用它，在《用户手册》中也会引用它
	* `conref` 片段级复用：**场景** - 产品名称（一改则全改），好比在 `settings.py` 中设置了一个全局变量 `APP_NAME = "SuperApp v1.0`，文档里不会出现名称本身，而是使用引用（占位符）。
* 条件处理（Profiling）≈ Feature Flags（功能开关）/ If-Else
	* DITA 的 `.ditaval`过滤文件，就相当于部署代码时使用的 `Environment Variables`，告诉编辑器（DITA-OT）现在是为哪个环境构建版本。
* 结构化约束（Structured Writing）≈ Pydantic Models (FastAPI)
	* **DITA 概念**：DITA 强制要求你必须按 `<task> -> <steps> -> <step>` 的结构写，不能乱写。可以理解为 DITA 定义了一套 Schema（DTD/XSD），就像你定义了一个 Pydantic Model：
```Python
		class Task(BaseModel):
		title: str
		shortdesc: str
		steps: List[Step] # 必须包含 Steps，且必须是 List
 ```
		
如果在 Task 里漏了 Step，Oxygen 会报错，就像 FastAPI 请求体不符合 Schema 会报错一样。这保证了数据的**类型安全**（文档结构安全）。

## Concept

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE concept PUBLIC "-//OASIS//DTD DITA Concept//EN" "concept.dtd">
<concept id="concept_vcs_3d5_vhc">
    <title></title>
    <shortdesc></shortdesc>
    <conbody>
        <p></p>
    </conbody>
</concept>
```
1. `<concept id="...">` —— 身份证号：建议使用包含语义的ID，而不是随机ID
2. `<title>` —— `<h1>` 标签，在 Concept 里，通常是名词或名词短语，区别于Task的动词短语
3. `<shortdesc>` —— Python Docstring / Meta Description，可以理解为 docstring，功能为：
	* 自动化生成：生成PDF或 WebHelp时，会自动出现在目录的章节标题下方，或者搜索结果的摘要里
	* 上下文预览：鼠标悬停在链接上是，显示的也是它。
4. `<conbody>` —— `<div class="container">`：在 Concept 的 `conbody`里，严谨出现 `<steps>`标签——讲原理的时候不要讲操作
5. `<p>` —— `<p>`
6. `<ul>` / `<ol>` / `<li>`
7. `<fig>` / `<image>`
8. `<section>`

## Task
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE task PUBLIC "-//OASIS//DTD DITA Task//EN" "task.dtd">
<task id="task_uyg_vw5_vhc">
    <title></title>
    <shortdesc></shortdesc>
    <taskbody>
        <context>
            <p></p>
        </context>
        <steps>
            <step>
                <cmd></cmd>
            </step>
        </steps>
    </taskbody>
</task>

```
### 1. `<context>` —— 场景/前置条件 (The Scenario)
- **位置**：它在 `<steps>` 之前。
- **作用**：告诉用户 **“在什么情况下执行这个任务”**。
- **例子**：_“当显示器电源指示灯闪烁红色时，请执行以下复位操作。”_
- **Developer Note**：
    - 这就像函数的 **Comments** 或者 **`if (condition)`** 描述。
    - **面试坑点**：不要在这里写步骤！不要写“第一步，拔掉电源”。这里只能写背景。
### 2. `<steps>` & `<step>` —— 算法循环 (The Algorithm Loop)

- **结构**：`<task>` 必须包含 `<steps>`（或者 `<steps-unordered>`，但很少用），`<steps>` 必须包含至少一个 `<step>`。
- **Developer Note**：
    - 这就是一个 **Ordered List (`<ol>`)**。
    - DITA 会自动帮你处理编号（1, 2, 3...）。**千万不要手动写 "1. Click OK"**，因为如果你中间插入一步，DITA 会自动重排数字，手动写就死定了。
### 3. `<cmd>` —— 核心指令 (Command / Executable Code)
- **地位**：这是 `<step>` 里 **唯一必须存在** 的子标签。没有 `<cmd>`，Oxygen 会报错。
- **写作原则**：必须是 **祈使句 (Imperative Mood)**。
    - _Good:_ "Click the **Save** button." (点击保存按钮。)
    - _Bad:_ "You should click the save button." (你应该点击...)
- **Developer Note**：
    - 这就是代码里的 **执行语句**（比如 `button.click()`）。它必须是一个具体的动作。

## Reference
```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE reference PUBLIC "-//OASIS//DTD DITA Reference//EN" "reference.dtd">
<reference id="reference_t1s_r1v_vhc">
    <title></title>
    <shortdesc></shortdesc>
    <refbody> </refbody>
</reference>

```
Reference ≈ 数据库 / API 文档
- 只有 Facts，没有 Story。
- 用户通常会做 Search 或 Lookup，不会全文阅读。
- SSOT = Single Source of Truth

### `<refbody>` 里装什么：结构化数据

最常用是：
- `<table>` (CALS Table)：非常复杂，支持合并单元格、自定义列宽。**这就是你在 OSD 模拟器项目里遇到的那种复杂 Excel 逻辑的归宿。**
  > 完全可以利用 Python 脚本将 Excel 自动转换为 XML 格式的 Reference 文件，以避免人工 Copy-Paste 产生的错误。
- `<simpletable>` 结构简单，类似 Markdown 的表格。适合写“错误码对照表”。
- `<properties>` 属性表，完全是 `JSON` 对象。

为什么不把数据写在 Task 里？
- 解耦
正确的示范：
`Step 3: Tighten the screw. Refer to **[Torque Specifications]** for details.`


## Topic

非常宽松的一种 DITA 文件类型。那为什么还需要它呢？
- 作为“容器”或“章节页”：可以想象为一个需要挂载子路由的 `<router-view>` 组件
- 无法归类的特殊内容：比如术语表、法律声明
除非你确定这块内容**既不是 Concept，也不是 Task，也不是 Reference**，否则不要碰 generic Topic。

## Map

如果没有 MAP，Concept/Task/Reference 只是硬盘里的一堆碎片。`.ditamap`是将这些碎片组装成一本书的“图纸”。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE map PUBLIC "-//OASIS//DTD DITA Map//EN" "map.dtd">
<map>
    <title></title>
</map>

```

DITA Map ≈ `router/index.ts` 或 `urls.py`。

* ***核心功能**：
	- Map 本身**不写正文**。
	- 它只负责 **“路由” (Routing)** 和 **“组织结构” (Hierarchy)**。
	- 它告诉编译器（DITA-OT）：先渲染哪个文件，后渲染哪个文件，谁是谁的子章节。
* **关键标签：`<topicref>` (Topic Reference)** 这是 Map 里最重要的标签。它的作用相当于 Python 的 `import` 或者 C 语言的 pointer。
	```XML
	<map>
	    <title>我的手册</title>
	    <topicref href="introduction.dita">
	        <topicref href="safety_warning.dita" />
	    </topicref>
	    <topicref href="installation_task.dita" />
	</map>
	```
虽然我们要看懂代码，但在实际工作中，**绝对没人手写 `<topicref>`**。
在 Oxygen 的 **DITA Maps Manager** 面板（通常在左侧）：
1. **操作**：你只需要从文件浏览器里，把刚才建好的 `concept.dita`、`task.dita`、`reference.dita` **拖拽 (Drag & Drop)** 到 Map 里面。    
2. **调整层级**：想让 Task 变成 Concept 的子章节？直接在 Map 面板里把 Task **拖到** Concept 上面去。
3. **效果**：Oxygen 会自动在后台帮你把 XML 代码改成嵌套结构。

### Map 的进阶功能：`<reltable>` (关系表) 

场景：假定你知道“凡是看 A 内容的人，就肯定会看 B”，你就可以将这种关系通过 `<reltable>`建立在 Map 里，这样编辑器会自动在 A 的底部生成一个指向 B 的链接。

对于复杂的文档，应该倾向于使用 Relationship Table 来管理链接，而不是在正文里 Hard-code `xref`。这样能保证链接的健壮性，而且能避免产生死链。

### Map 的过滤功能

同一个 Map，加上一个 `<val>` 过滤文件，今天生成“内部验证版 PDF”（包含 Reviewer Notes），明天生成“客户交付版 PDF”（隐藏敏感信息）。