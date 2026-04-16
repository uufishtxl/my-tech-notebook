# DRF Serializer 生命周期与读写分离实战

> [!NOTE]
> 掌握序列化（Read）与反序列化（Write）的对称过程，理解数据在不同阶段的状态转移。

---

## 1. 核心心智模型：读写生命周期对比

| 阶段 / 属性 | 📥 方向一：反序列化 (Write / 入栈) | 📤 方向二：序列化 (Read / 出栈) |
| :--- | :--- | :--- |
| **起点 (入口)** | 外部输入的 **原生 JSON / FormData** | 打包好的 **Model 实例 (或 QuerySet)** |
| **第一步：装载** | `serializer = get_serializer(data=request.data)` | `serializer = get_serializer(instance)` |
| **第二步：校验** | 必需执行：`serializer.is_valid()` | 无此步骤（数据库数据默认为合法） |
| **数据形态** | 清洗结果存放在 `serializer.validated_data` (Dict) | - |
| **第三步：拦截** | `perform_create` 钩子中进行手动“补足” (如 user) | 调用 `.data` 属性触发延迟计算/转化 |
| **第四步：保存** | `serializer.save()` 真正执行 ORM 入库 | - |
| **终点 (出口)** | 数据库中的 **Model 实例** | 返回前端的 **JSON 字符串** |

---

## 2. 核心难点：read_only_fields 与数据库约束

### 2.1 闭环困境
如果一个字段是数据库 **必填 (NOT NULL)**，但在 Serializer 中标记为 **只读 (Read Only)**：
1. `is_valid()` 会直接扔掉前端传来的该字段值。
2. 导致 `validated_data` 缺损。
3. `serializer.save()` 会引发数据库异常（IntegrityError）。

### 2.2 解决方案
必须在 ViewSet 的 `perform_create` 或 `perform_update` 钩子中进行“手工补完”：
```python
def perform_create(self, serializer):
    # 手动从 request 中拿取身份信息填补 readonly 的空缺
    serializer.save(user=self.request.user)
```

### 2.3 `HiddenField` 方案
若想彻底摆脱 ViewSet 的覆写，可在 Serializer 内使用：
```python
user = serializers.HiddenField(default=serializers.CurrentUserDefault())
```

---

## 3. 读写分离（海关清单 Meta）
- **`model`**：货品图纸（ORM 模型）。
- **`fields`**：白名单，只有名单内的能通关。
- **`read_only_fields`**：严禁进口（前端不可改，仅由后端生成，如 id, created_at）。

---

*创建日期: 2026-02-28*
