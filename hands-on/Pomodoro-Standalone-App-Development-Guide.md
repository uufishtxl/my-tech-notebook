# Pomodoro Standalone App 开发全流程指南

针对番茄钟应用从 `reader` 剥离为独立 `standalone app` 的实战大纲。

---

## 第一阶段：骨架搭建 (Models)
**重点：定义出数据结构和“状态”**
1.  **用户关联**：别忘了每个专注记录都应该属于一个 `User` (ForeignKey)。
2.  **标签系统 (`PomodoroTag`)**：
    - `name` (CharField)
    - `order` (IntegerField) - 用于前端排序。
3.  **核心记录 (`Pomodoro`)**：
    - `user`, `tag` (ForeignKey)。
    - `duration` (IntegerField) - 专注时长。
    - `status` (CharField) - **核心实战点**：使用 `choices` 定义有限状态机（如：`STARTED`, `COMPLETED`, `INTERRUPTED`）。
    - `created_at` (auto_now_add)。
    - `completed_at` (DateTimeField, null=True) - **修正逻辑**：不要用 `auto_now_add`，这个时间应该在任务真正结束时通过 API 手动填入。

## 第二阶段：契约定义 (Serializers)
**重点：处理嵌套和只读安全**
1.  **`PomodoroTagSerializer`**：简单的字段定义。
2.  **`PomodoroSerializer`**：
    - **嵌套展示**：把 `tag` 嵌套进来显示名字，而不是只给一个 ID。
    - **安全保护**：把 `id`, `created_at`, `completed_at`, `status` 设为 `read_only_fields`。这些神白的时间戳和状态不应该允许前端通过 POST 乱改。

## 第三阶段：门户开放 (Views & URLs)
**重点：利用 ViewSet 的高效率**
1.  **`ModelViewSet`**：利用 `viewsets.ModelViewSet` 快速搞定 CRUD。
2.  **权限控制**：确保 `permission_classes = [IsAuthenticated]` 且 `get_queryset` 只能查到当前用户的数据。
	TIPS: 查询当前用户数据，只需要覆写 `get_queryset`，因为在 ModelViewSet 中，无论是单条数据的获取 (`retrieve`)、更新 (`update`) 还是删除 (`delete`)，内部都调用一个核心方法：`self.get_object`，而`get_object` 源码逻辑大致如下：
	- 先调用 `self.get_queryset()`拿到基础数据集
	- 在这个数据集基础上，根据 URL 里的 `lookup_field`（通常是 `pk`）进行过滤
3.  **动作定义 (Custom Actions)**：
    - **实战点**：在 `PomodoroViewSet` 里通过 `@action` 定义扩展端点。
	    - [x] **示例 1：`ongoing`**：用于检查是否有正在进行的专注任务。
	    - [x] **示例 2：`history`**：用于获取特定日期的记录，并按需进行升序排列。
    - **提示**：原本建议的 `finish` 动作已被更优雅的 `perform_update` 钩子取代。
4.  **路由注册**：在 `pomodoro/urls.py` 里用 `SimpleRouter` 注册，并挂载到主项目的 `api_v1_urls.py`。

## 第四阶段：后勤保障 (Admin)
**重点：监控与校验**
1.  **可视化**：配置 `list_display` 让你在后台一眼看到哪些计时正在进行。
2.  **只读显示**：利用 `readonly_fields` 让自动生成的时间戳可见。

---

## 💡 踩坑笔记 (Hands-on Pitfalls)
在实战过程中总结的宝贵经验：

1. **`completed_at` 的逻辑陷阱**：
   - 错误：设置为 `auto_now_add=True`。
   - 结果：导致结束时间永远等于创建时间。
   - 正解：应设置为 `null=True, blank=True`，由业务逻辑在结束时手动填入。

2. **级联删除的副作用 (`models.CASCADE`)**：
   - 在使用外键时，务必思考：删掉父项（如 Tag）时，是否真的希望子项（如 100 条专注记录）也被瞬间抹去？
   - 保护策略：考虑使用 `on_delete=models.PROTECT`。

3. **模型层的数据校验**：
   - 错误：试图在字段定义中使用 `gt=0`。
   - 正解：Django ORM 不支持直接的数学比较参数，需使用 `from django.core.validators import MinValueValidator`。

4. **反向查询的优雅性**：
   - 在定义 `ForeignKey` 时，习惯性思考是否需要 `related_name`。这能让你在从用户或标签出发查询相关番茄钟时，代码更加自然（如 `user.pomodoros.all()`）。

5. **非空字段的迁移策略**：
   - 在向既有数据的表中新增非空字段（如 `user`）时，为避免迁移报错，可以暂时设置一个 `default=1`，或者在迁移提示中提供一次性默认值。

6. **User 模型的“身份迷思” (`Settings` vs `get_user_model`)**：
   - **坑点**：在 Shell 中直接使用 `settings.AUTH_USER_MODEL.objects.all()` 时报错 `AttributeError: 'str' object has no attribute 'objects'`。
   - **原因**：`settings.AUTH_USER_MODEL` 在配置文件中只是一个普通字符串，它不是真正的类。
   - **正解**：必须使用 `from django.contrib.auth import get_user_model; User = get_user_model()`。

---

## 🚀 [实战进阶] ViewSet 自动化与实时同步

### 1. `perform_create` —— 创建时的自动补全 + 安全检测
```python
def perform_create(self, serializer):
    existing = Pomodoro.objects.filter(user=self.request.user, status='started').first()
    if existing:
        existing.status = 'interrupted'
        existing.save()
    serializer.save(user=self.request.user)
```

### 2. `perform_update` —— 状态变更时的自动打卡
```python
def perform_update(self, serializer):
    instance = self.get_object()
    new_status = serializer.validated_data.get('status')
    if new_status == 'completed' and not instance.completed_at:
        serializer.save(completed_at=timezone.now())
    else:
        serializer.save()
```

---

## 🧩 [实战进阶] Serializer 读写分离与海关模型

### 1. "读写分离"的优雅技巧 (Nested Serializer + PrimaryKeyRelatedField)
- **只供写（专属投币口）**：`tag_id = serializers.PrimaryKeyRelatedField(queryset=PomodoroTag.objects.all(), source='tag', write_only=True)`
- **只供读（豪华展示柜）**：`tag = PomodoroTagSerializer(read_only=True)`

### 2. 核心比喻：DRF 海关清单 (Meta)
- **`read_only_fields` (严禁进口清单)**：标识为只读后，即便前端 POST 了这些值，海关也会直接扔掉并不报错。

---

## 💡 核心技巧：Django 中的日期与时间处理 (Tips & Tricks)

1. **ORM 的“手术刀”过滤**：`created_at__date=date_str`
2. **获取当前时间**：永远使用 `timezone.now()`
3. **时间加减法**：`timedelta`
4. **原则**：**存储用 UTC，显示用本地。**

---

## 💡 深入理解：DRF 视图机制与 Python 面向对象

在重构和梳理视图时，理清这几个底层运转机制：

1. **`self.request` 的挂载机制（`APIView`）**
   - 为什么 `get_queryset` 里面可以直接写 `self.request`？因为 DRF 在 `dispatch` 时已自动挂载。

2. **`@action` 方法的签名规范**
   - 自定义视图必须显式接收 `request` 参数，且需手动返回 `Response()` 对象。

3. **模型内部枚举（如 `Status`）与私有属性**
   - Python 私有化惯例是加单下划线 `_`，双下划线 `__` 会触发 Name Mangling 机制。

4. **`get_serializer` 与 `.data` 的执行时机**
   - 只有在显式访问 `.data` 属性的那一刻，才会真正执行序列化计算。

---
