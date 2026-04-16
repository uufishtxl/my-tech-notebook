# Audio Slicer - DRF 重构实战演练指南 (Hands-on)

> [!NOTE]
> 本指南旨在通过重写 `audio_slicer` 模块的 DRF 视图和序列化器，全面检验和巩固在 `reader` 模块中学习到的高级 DRF 技巧（生命周期钩子、自定义 Action、文件处理和外部服务集成）。

---

## 阶段零：环境准备与防干扰

为了不破坏现有的可用代码，我们采取**平行重构**的策略。

1. **创建新沙箱应用**：
   在 Django 中创建一个新的 app，例如 `audio_slicer_v2`。
   `python manage.py startapp audio_slicer_v2`
2. **复用模型**：
   **不要**在 `audio_slicer_v2` 中重新定义 Models。直接在你的新序列化器和视图中，导入原本 `audio_slicer.models` 里定好的 `SourceAudio`, `AudioChunk`, `AudioSlice` 等模型。
3. **注册新路由**：
   在主路由 `api_v1_urls.py` 中，将新 app 挂载到一个专门的测试前缀下，比如 `/api/v2/audio_slicer/`，以便于使用 Postman/前端 独立测试。

---

## 阶段〇：模型层 (Models) 的底层防御与约束

**核心目标：** 在模型层构建健壮的防线，避免在业务层产生意想不到的级联销毁或查询死锁。

### 踩坑记录与新知验证 (来自 Playground Sandbox)
1. **致命的 `CASCADE` 级联删除隐患**
   - **错误场景**：如果在 `Drama` 和 `SourceAudio` 模型中，`user` 外键使用了 `on_delete=models.CASCADE`。那么当某个员工离职被注销账号时，他当年上传的所有独一无二的公共大局（如《老友记》第一季）及底下的所有音频、笔记将全部瞬间灰飞烟灭！
   - **正确做法**：将资源型数据的拥有者改为 `on_delete=models.SET_NULL, null=True, blank=True`（人走茶不凉，转交归属或留空为 Anonymous），或者 `on_delete=models.PROTECT`（强行阻止删人，必须先转移遗产）。

2. **动态默认值 (Dynamic Defaults) 与 `save()` 重写**
   - **坑点**：你想在 `SourceAudio` 的 `title` 字段上设置 `default` 属性，让它根据 `drama` 和 `season` 自动拼接取名。但发现 Django 字段的 `default=func` 中，函数内部是拿不到 `self` (当前实例) 的！
   - **正确做法**：只有重写 `def save(self, *args, **kwargs):` 才是这种动态组合字段唯一合法的救赎之道。
   它会且仅会发生在：**当这个对象即将被真正写入数据库的那一瞬间**。
	
	具体来说，在整个数据流转环节里，它发生得非常靠后（是进数据库前的最后一道守门员）：
	
	1. **前端发送请求**: `POST {"drama_id": 1, "season": 1, "episode": 1, "title": ""}`
	2. **DRF Serializer 解析 & 验证**: 检查传过来的数据类型对不对（比如整数）、有没有违反唯一性约束。这时候 save() 还没执行。
	3. **调用 `serializer.save()`**: 验证通过后，准备落库。
	4. **👉 触发模型的 save() 钩子**: 就在这个时候，Django 要把对象拼成真正的 SQL 语句发送给数据库之前，它看到了你写的自定义 save() 方法。
	    - 它进去检查发现 `self.title` 是空的（或者只是个在验证期就空着的空字符串 `""`）。
	    - 于是，它当场把 `self.drama.name` 和季数集数拼好，强行赋值给 `self.title`。
	5. **执行 `super().save(*args, **kwargs)`**: 这句话就是真正触发 Django 底层的 `INSERT INTO ...` 或者 `UPDATE ...` 的 SQL 语句，把带有刚拼好的标题的数据存进硬盘。
   
6. **隐式的反向查询陷阱 (`_set` vs `related_name`)**
   - **现象**：即使以前没有在 ForeignKey 上写 `related_name="audios"`，也依然能查到剧集下的音频。
   - **揭秘**：因为 Django 底层贴心（又丑陋）地自动生成了 `小写模型名_set` (如 `sourceaudio_set`)。但为了代码的极致优雅和防冲突，永远记得显式地定义 `related_name="audios"`（这样就可以用 `drama.audios.all()` 啦）。

4. **次世代约束 `UniqueConstraint`**
   - 老古董 `unique_together` 只能把几个字段打包干巴巴地去重。
   - 现代版防线：使用 `Meta.constraints = [models.UniqueConstraint(fields=[...], name='...')]`。最大的魅力是可以加 `condition=Q(is_deleted=False)` 等**条件去重**（局部索引约束），直接与现代关系型数据库的底层高阶特性接轨。

5. **工业级路径清洗 (Regex Regex Regex!)**
   - **痛点**：用户填装的 `name` 千奇百怪，包含特殊符号。如果只用 `.replace(' ', '_')`，它在碰到包含例如冒号 `:`、问号 `?` 甚至是斜杠 `/` 时会触发操作系统层面的写入异常或者 URL 路由灾难。
   - **绝杀手段**：在 `upload_to_func` 内必须使用正则暴力清洗法：`re.sub(r'[^\w\s-]', '', source.drama.name).strip().replace(' ', '_')`。

6. **动态钩子的底层参差 (`upload_to` vs `default`)**
   - **神奇的 `upload_to`**：为何 `upload_to=my_func` 可以跑，而且能接收参数？因为在文件落盘的一刹那，Django 给它优待传入了当前已被实例化完整的 `(instance, filename)`。我们在内部可以大摇大摆调用 `instance.drama.name`。
   - **拉跨的 `default`**：而给普通字段设置 `default=my_func` 时，它被调用得极其早，根本不会带任何上下文参数 `()`。这就是为什么想要结合其他字段得出动态 default，只能老老实实去重写 `save()`。

7. **时间字段的语义与防篡改 (Timestamps)**
   - `auto_now_add=True` (出生证明，仅第一次有效)。
   - `auto_now=True` (变动追踪，一旦调用 `save` 就强制更新为此刻)。
   - **业务驱动型时间**（例如：`last_studied_at`）：坚决不能用上述两项。必须设置 `null=True, blank=True`，交给具体的业务试图在完成操作时，使用 `timezone.now()` 服务器绝对权威时间手动赋值，防止前端传伪造的过期时间回来。

8. **枚举类的现代化表达 (`models.IntegerChoices`/`TextChoices`)**
   - 将原古董元组 `REVIEW_TYPES = (('translation', 'Trans...'), ...)` 彻底替换。
   - **安全性**：这只是一种代码/框架验证层的“软约束”，用来给后端带来极致的代码提示体验和过滤把控。它在真正的底层数据库里依然是单纯的 INT 或 VARCHAR 字段，没有任何物理破坏性。

---

## 阶段一：序列化器 (Serializers) 深度复刻

**核心目标：** 掌握嵌套读取、字段保护与自定义校验。

### Task 1.1: 基础属性与 `read_only_fields`
- **目标：** 为 `SourceAudio` 和 `AudioChunk` 编写序列化器。
- **要点：** 
  - 思考哪些字段由用户提交（如 `title`, `audio_file`），哪些是由后端生成的（如 `duration`, `status`）。
  - 使用 `read_only_fields` 保护后端生成的字段，防止被恶意用户在 POST/PUT 时篡改。
- **验收：** 确保 POST 创建时，尝试传入受保护的字段不会生效。

### `SourceAudio`
用户上传的时候，提供：
- Drama
- Season
- Episode
- multi-fromat

### Task 1.2: 嵌套的序列化器 (Nested Representation)
- **目标：** 在获取某一集的 `SourceAudio` 详情时，把该集下面所有的 `AudioChunk` 一并返回。
- **要点：** 
  - 在 `SourceAudioSerializer` 中，利用 `SerializerMethodField` 或者直接嵌套 `AudioChunkSerializer(many=True, read_only=True)`。
- **验收：** GET 一条 `SourceAudio` 详情，JSON 结构中包含 `chunks` 列表。

---

## 阶段二：视图集 (ViewSets) 与生命周期钩子

**核心目标：** 掌握权限隔离、创建前后的逻辑注入。

### Task 2.1: 基础 CRUD 与数据隔离
- **目标：** 使用 `ModelViewSet` 实现 `SourceAudioViewSet`。
- **要点：**
  - **权限**：确保只有带 Token 的用户才能访问 (`IsAuthenticated`)。
  - **隔离 (重写 `get_queryset`)**：确保用户只能看到和操作 **自己的** 录音数据。
- **验收：** GET 列表接口只返回当前用户的记录；无法 GET 到别的用户的数据。

### Task 2.2: 拦截与外部服务 (重写 `perform_create`)
- **目标：** 当用户上传一个全新的录音文件时，保存记录，并触发外部的切片任务。
- **要点：**
  - 拦截默认行为，使用 `serializer.save(user=self.request.user)` 绑定所属用户。
  - 在 `save()` 执行**之后**，调用外部函数（参考原来的 `slice_source_to_chunks(instance)`）。
- **验收：** POST 成功创建记录后，外部处理函数被正确触发。

---

## 阶段三：灵活使用 `@action` 装饰器

**核心目标：** 跳出 REST 标准模式，实现高度定制化的业务接口。

### Task 3.1: 文件上传的自定义 Action (`detail=True`)
- **目标：** 为具体的 `SourceAudio` 实现一个上传封面的接口。
- **要点：**
  - 使用 `@action(detail=True, methods=['post'])`。
  - 这个接口只接收和处理文件 (`request.FILES.get('cover_image')`)，不需要走全套的序列化验证。
- **验收：** POST `/api/v2/audio_slicer/source_audios/{id}/upload_cover/` 成功修改封面图片并保存。

### Task 3.2: 业务逻辑推断与组合 (`detail=True`)
- **目标：** 实现 `AudioChunk` 的 `complete` 功能。
- **要点：**
  - 用户提交完成当前片段。
  - **后端计算**：将当前片段标为“已完成”，并自动去数据库里查找本集 (SourceAudio) 的**下一个**片段的 ID 并一并返回。
- **验收：** 接口不仅更新了 DB，还返回了诸如 `{ "next_chunk_id": 45, "is_last": false }` 这样高信息量的 JSON 供前端导航。

### Task 3.3: 批量操作 (`detail=False`)
- **目标：** 实现 `AudioSliceViewSet` 的批量翻译功能。
- **要点：**
  - 这是一个针对集合的操作：`@action(detail=False, methods=['post'])`。
  - 接收一个 ID 数组，去数据库里查出这些对象，调用外部 AI 翻译服务，最后批量更新保存 (`bulk_update` 或循环 `save`)。
- **验收：** 发送 `{"ids": [1, 2, 3]}`，后端成功将这 3 个片段更新上了翻译内容。

---

## 总结与自查

当完成上述重构后，请对照原版的 `audio_slicer/views.py` 与你写的版本：
1. 我的代码是不是比原版更**薄**（序列化器承担了数据校验，视图只管调度）？
2. 权限是不是真正做到了滴水不漏？
3. `@action` 的运用是不是合理剥离了非典型的 CRUD 业务？
---

## 阶段四：网络请求与 API 设计思考 (API First / Contract Driven)

这次重构也带来了一些高级的前后端交互经验与接口设计思维模型。

### 1. 如何甄别使用 POST / PUT / PATCH：不要被语义迷惑
- **误区**：很多人认为 POST 就是“新建记录”，PUT就是“全部替换”，PATCH就是“局部更新”。在某些动作型场景（如 Submit Review Card）或批量操作场景时，往往会因为“这是在更新状态”而死磕用 PUT/PATCH。
- **正解**：永远别被英文单词表面的语义绑架，一切以 **HTTP 幂等性（Idempotency）**和业务副作用为准：
  - **PUT 是幂等的**：意味着你无论发送多少次相同的请求，资源最终的结果必须是绝对静止的同一张快照。它主要用于单体资源的“完全覆盖”。
  - **PATCH 虽然常用于更新属性，但也应对局部状态明确变动**。 
  - **POST 的两大高阶使用场景（破除 CRUD 迷信）**：
    1. **非幂等的业务动作**：适合用于“动词导向”的操作或带有持续副作用累加的行为（例如 `POST /submit/`，它不仅改了卡片的 Box Level，还会推进下一次复习时间。前端若在不小心短时间内发了两次请求，业务上它是往后发生累加的，因此必须用非幂等的 POST 来明示副作用危险）。
    2. **超出单体资源 CRUD 语义的批量操作 (Batch / Upsert)**：即使后端的批量更新在处理上确实是幂等的（例如基于 chunk 坐标的 `update_or_create`），但由于向集合路径发送 PUT（如 `PUT /audioslices/`）的严格 REST 语义是“用提交的 payload **完整覆盖替换掉**服务器上的整个集合”，为了避免这种灾难级的语义混淆，业界统一的最佳实践是将批量操作 / 自定义 Action 降级，采用万能的 POST 作为调用载体（如 `POST /audioslices/create_batch/`）。

### 2. 破除嵌套，将 ForeignKey 关联的数据扁平化呈现给前端及适用考量因素
- **问题场景**：在传统方案中，`ReviewCardSerializer` 经常抛出完整的 `audio_slice` 嵌套结构树。这不仅导致前端更新原文字段时需深层寻址，更加剧了前后端联调的认知成本。
- **最佳实践：扁平化视图模型 (Flatten View Models)**
  - **后端巧招**：在 Serializer 层利用 `source='audio_slice.original_text'` 等参数彻底将其抽平，使其在前端接收时成为同级并列的干净字段。前端只需获取 `slice_text`, `slice_translation`, `audio_url`，不再陷入深层次解构噩梦。
  - **适用此类场景的考量因素**：
    1. **只读聚合层**：如果是纯展示复合卡片列表，扁平化是性能与开发体验的双赢。
    2. **读写分离定律**：前端若需要修改底层的 `AudioSlice` 业务（如修改字幕、修复翻译），那么上层聚合用的 `ReviewCardSerializer` 中这部分抽平的字段应坚定标记为 `read_only=True`！然后，暴露专属于底层实体的原生接口（例如 `PATCH /v1/audioslices/{id}/`）供前端精准提交修改，**杜绝通过顶层嵌套来强行深穿写入的高耦合写法**。这种“呈现聚合抽平，修改底层直达”的模式是系统解耦的黄金准则。
	    ↑↑↑ 非常重要 

### 3. 工业级的闭环流程（API First / Contract Driven）
- **合并零碎的请求**：发现针对同一实体 `AudioSlice` 的局部变更（如更新原文和更新翻译），如果本身只是 Payload 的局部变化，就应当合并为一个强大的 `PATCH` 接口！不要基于前端动作无限生化出零碎重复的 update 命名方法。
- **契约驱动（Contract Driven）的降维打击**：
  - **API First**：不再是后端写什么样的数据结构就随便往外抛什么，也不让前端看着一堆嵌套迷茫。前后端基于 Interface 与契约参数共同商议后并形成规矩，例如通过统一规整一份强类型的 `reviewApi.ts` 来推进。
  - 以后端的数据源为例，就算后端数据库结构变迁或者拆表，它对外都必须严格履行定好的 Contract 拍扁并映射出去。这样后端改内部逻辑，对前端完全无痛且静默。

### 4. 接口响应契约与序列化器的解绑

在真实世界的复杂业务中，当我们执行一个**动作型（Action / Mutation）**请求时（比如调用了 `@action(detail=True, methods=['post']) def submit(self, request, pk=None):`），后端的目的不再是“获取最新信息的快照展示给用户看”，而是：

1. **触发内部复杂的业务流转**（在这里是跑一遍 Leitner SRS 算法更新 Box 属性和下次复习时间戳）。
2. **立刻只返回前端关心的“动作执行结果”**，用于前端做出反馈式的微小 UI 更新（例如前端弹出提示框：“升级至 Box 3！”）。

后端这里**根本就没有调用任何 Serializer**（比如 `serializer.data`），而是直接硬编码拼装了一个轻量级的字典吐回去了！

#### 这种设计的意义：动作结果反馈

- **避免浪费带宽计算**：前端其实只需要知道自己的点击有没有生效（`status`）、卡片升级到了哪里（`new_level`），完全没必要让后端再去查一次完整的连带 AudioSlice 和巨大原文句子的长文本重新走一遍 ReviewCardSerializer。
- **Contract Driven（契约驱动）**：前端 TS 类型中写的这个 ReviewSubmissionResult，就是跟后端约定好了的一纸契约——“你按了提交，我就只精准返还这三个提示数据，不啰嗦”。

---

### 5. DRF 序列化器字段类型与隐藏特性

在序列化器的编写过程中，有一些很容易踩坑的字段映射类型与隐藏特性：

- **Model TextField vs DRF CharField**：
  Django Model 中存在 `models.TextField()` 用来存储毫无长度限制的长文本。但 **DRF 并没有 `TextFieldSerializer` 这种东西！**在 DRF 序列化器中，无论对应的 Model 是普通的 `CharField`还是长文本，都应该统一使用 `serializers.CharField()`。

- **FileField/ImageField 自动提取 URL 的魔法**：
  当你在 Serializer 中提取底层模型的文件时，你不需手动去写转换逻辑。**DRF 的 FileField 在序列化时会自动根据文件的 `.url` 属性，返回完整可访问的 URL**！只要配合包含 request 的 context（如 ModelViewSet），最终吐给前端的就是携带域名的绝对路径（如 `http://localhost:8000/media/...`）。

---

祝重构愉快！随时记录踩坑心得！

---

## 阶段五：进阶避坑实录（来自 Playground Sandbox）

### 1. ORM 过滤的“指鹿为马”陷阱 (`id` vs `外键_id`)
- **情景**：在使用 `SourceAudio.objects.filter(id=drama_id)` 检索属于某个 `Drama` 的所有音频时，没有拿到预期数据。
- **解剖**：因为这里的 `id` 指代的是 `SourceAudio` 表自身的自增主键（即这单条语音的 ID）。Django ORM 不会自动猜测你指的是它关联的 `Drama` 主键。
- **正确姿势**：应对外键查询有两种安全解法：
  - **写法 A**（只拿到了外表 ID 值）：`filter(drama_id=drama_id)`，利用 Django 隐式生成的关联字段名。
  - **写法 B**（手握外表模型对象）：`filter(drama=drama_obj)`，直接传递实例对象，更加优雅防错。

### 2. `@action` 装饰器的核心口诀 (`detail=True` vs `detail=False`)
- **误区**：以为只要是在针对该 Model（比如 `SourceAudio`）的接口，方法就应该设为 `detail=True`。
- **口诀：这个请求，到底需不需要知道当前这张表里的一条具体数据的主键（pk）？**
  - **需要 (`detail=True`)**：例如给某一集上传封面 (`upload_cover`)，路由 `.../audios/{pk}/upload_cover/`，必须由 URL 锁定主键。
  - **不需要 (`detail=False`)**：例如获取某个剧集下的所有集数集合 (`episodes`)，路由 `.../audios/episodes/?drama_id=xxx`。这种通过 `query_params` 筛选出一个合集/列表的操作，并不依赖具体哪一条录音的 `pk`，因此绝不能加 `detail=True`。

### 3. 防御性编程的边界识别 (防御过度)
- **情景**：在获取 `seasons = ...values_list('season')` 后，在转换列表渲染时写了列表推导式防御空值：`[s for s in list(seasons) if s is not None]`。
- **真相**：对于是否需要防御过滤，永远以 **Database Schema (Model 约束)** 为主源基准。观察模型定义 `season = models.PositiveIntegerField()`，这里没有声明 `null=True, blank=True`。
- **结论**：既然 DB 约束这个字段绝对必填 (NOT NULL)，就意味着提取出来的值绝不可能为 `None`。因此无需防御 `None`，可以直接放心地返回干净利落的 `list(seasons)`，让代码表达保持极致的纯净。

