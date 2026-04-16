# DRF 视图与序列化器实战心法：POST 请求的一生

> [!NOTE]
> 记录 DRF 处理 POST 请求时的三个关键关卡，以及何时、如何去重写它们。同时包含了视图与序列化器的编写思路。

---

## 1. `POST` 请求在 DRF 中的一生

一个典型的创建操作（`POST`）在 DRF `ModelViewSet` 中会经历以下三道关卡：

### 第一道关卡：`get_serializer_class(self)` —— “选图纸”
系统要做的第一件事是：**“我该拿哪张安检图纸来检查这批货？”**
- **常态**：默认情况下，它会去读你写在顶部的 `serializer_class = ArticleSerializer`。
- **何时重写它**：如果你想做到**“因地制宜”**（比如“不对称序列化”）。当动作是 `create` 时，你让它返回 `ArticleCreateSerializer`；当动作是 `list` 时，返回 `ArticleListSerializer`。

### 第二道关卡：`create(self, request, *args, **kwargs)` —— “车间主任（总包工头）”
这是处理 `POST` 请求的**绝对中枢**。它的默认物理源码极其干净，严格按顺序干了四件事：
1. **拿图纸**：`serializer = self.get_serializer(data=request.data)`
2. **过安检**：`serializer.is_valid(raise_exception=True)`
3. **下车间**：`self.perform_create(serializer)` （把通过安检的数据交给小弟去组装入库）。
4. **打包发货**：拿到成果，拼装 HTTP 201 响应和 Headers，返回给前端。

**何时重写 `create`？** 当你需要“**破坏或重组整个车间流水线**”的时候！
例如：入库前要清洗 HTML，入库后还要去查关联数据（如 `Paragraph` 批量入库），最后还要**临时换一个豪华序列化器**（如 `ArticleDetailSerializer`）来打包发货。这种跨越了安检、入库、打包全流程的“大手术”，**必须**由车间主任 `create` 亲自来重写。

### 第三道关卡：`perform_create(self, serializer)` —— “流水线装配工”
这个钩子的默认源码极简，只有一行代码：
```python
def perform_create(self, serializer):
    serializer.save()
```
- **物理动作**：它不管安检，也不管发货，工作就是**按下 `save()` 按钮，把数据砸进数据库**。
- **何时重写它**：当流水线完全正常，你只需要在最后存入数据库的一瞬间，**偷偷塞进去一点只有后端知道的“私货”**（比如当前登录的用户、后台自动生成的处理状态）。
```python
def perform_create(self, serializer):
    # 拦截掉默认的 save()，塞入私货
    serializer.save(user=self.request.user, status="PROCESSING")
```
在这里写，是最轻量、最优雅的，因为你不需要去重新写那一长串的 `is_valid()` 和 `Response()` 返回逻辑，DRF 会帮你包揽其余工作。

---

## 2. 区分重写生命周期钩子还是使用 `@action`

下一次你在决定要重写哪个东西时，只问自己一个极其简单的物理问题：

- **我要无中生有，建一条新数据吗？**
    $\rightarrow$ 乖乖去走主视图集，用 `POST /xxx/`，重写 `perform_create` 或 `create`。
    
- **我要对某个已经存在的数据，触发一个特殊动作（比如：发邮件、调 AI、审核通过）吗？**
    $\rightarrow$ 永远使用 `@action(detail=True)` 另起炉灶，专事专办，做成 `/xxx/{id}/do_something/`。

---

## 3. The Secret Sauce: 编写序列化器的 Step-by-step 打法

1. **定契约**（考虑前端需要什么样的数据，用 JSON 描述出来）
2. **分读写**
	1. 如果既要读，又要写，建议直接读写分离。
	2. 遇到外键（像 Tag），果断拆成两行：一行 `tag` 给读（嵌套/深度丰富），一行 `tag_id` 给写 （`PrimaryKeyRelatedField`）。
3. **搭骨架**（填原生字段）
	稳住 `Meta` 基本盘：对应的 `model`，和白名单 `fields`。
4. **造骨肉**（处理“定制化”字段） 
	1. 缺前端需要的特殊文字展示？加上 `SerializerMethodField`。
	2. 缺复杂的嵌套关联表？加上嵌套的 Serializer（如 `ParagraphSerializer(many=True)`）。
5. **设安全**（保护字段规则）
    1. **法则1 (DRF 自动保护)**：主键、自动填写的时间戳(`auto_now_add`)、只读的关系字段。
    2. **法则2 (业务强制保护)**：绝对不能由前端随便通过 POST 修改覆盖的字段（例如：`ai_response`, `status` 等）。必须将其加入 `read_only_fields` 列表。

---

## 4. 获取 Request 数据的最佳实践

`get_queryset()` 方法调用时通常没有传入 `request` 参数。因为 DRF 收到请求后，会在准备分发方法前将 `request` 写入到 ViewSet 实例的属性上：`self.request`。

**最佳实践**：
- 如果一个方法签名里有 `request` 参数（如 `list(self, request)`、`create(self, request)`），直接使用局部变量 `request`，清晰且快速。
- 如果签名没有传这个参数（如 `get_queryset(self)`、`get_serializer_class(self)`），则使用 `self.request` 获取。

---
*整理自：Reader 模块挑战实录*