# 后端复习笔记


## 2026-01-25 测试记录 (5+5+5 Session)

### ✅ 已掌握
- **Durability (D)**：清楚理解断电/宕机后数据持久化的含义。
- **Python *args/**kwargs**：完全掌握 Tuple/Dict 的打包逻辑。
- **Shallow vs Deep Copy**：深刻理解嵌套列表在 copy() 下的行为。
- **Vue ref/.value**：清楚 JS 与 Template 的差异。
- **DITA Warning vs Caution**：准确区分人身伤害与设备损坏。
- **Pathlib rglob**：理解递归搜索子目录。

### 🔄 仍需加强 (待下次考)
- **[DRF] SerializerMethodField**：概念对了（类比 Vue computed），但名字拼写不熟。
- **[SQLAlchemy] joinedload vs selectinload**：由于长期用 Django，对 SQLAlchemy 的对应名词容易遗忘。
- **[Network] Timeout**：虽然这次答对了 Read Timeout，但可以再稳固下。
- **[Pathlib] path.parts**：昨天没看，今天跳过了。
- **[Performance] Throttle vs Debounce**：概念有点记混（例子举反了）。


## 2026-01-24 测试记录 (5+5+5 Session)

### ✅ 已掌握
- **N+1 查询问题**：清楚理解原理（1次查主表 + N次查详情）和解法（`select_related`/`prefetch_related`）。
- **RAG Retriever**：正确理解其在 LLM 之前检索上下文的作用。
- **Dataclasses default_factory**：理解为何 List 需要用 factory。
- **Vue Reactivity**：清楚 `ref` 在 template 自动解包，JS 中需 `.value`。
- **v-if vs v-show**：理解 DOM 销毁 vs CSS 隐藏的区别。

### ❌ 待加强 / 新增盲点
- **[DRF] 动态字段**：不知道 `SerializerMethodField`，容易想不起怎么加 Model 里没有的字段。
- **[Python] *args vs **kwargs**：混淆或不确定。`*args`=Tuple, `**kwargs`=Dict。
- **[Python] Shallow Copy**：误以为 `.copy()` 后修改内部嵌套对象（如列表中的列表）主对象不会变。
- **[Network] Timeout**：不清楚 Connection Timeout (接通前) vs Read Timeout (接通后等待响应) 的区别。
- **[Streaming] 内存差异**：主要在于 `StreamingHttpResponse` 使用生成器节省服务器内存，而非一次性生成。


## 2026-01-23 测试记录

### ✅ 已掌握
- 幂等性：GET/PUT/DELETE 幂等，POST 不幂等
- GIL：I/O 密集型影响小，CPU 密集型影响大
- async/await：单线程并发处理多个 I/O

### 🔄 仍需复习
- ACID 的 I (Isolation 隔离性) 和 D (Durability 持久性)

---



### ✅ 2026-01-23 已掌握
- ACID 的 I = Isolation（隔离性）
- position: sticky 原理（relative → fixed）
- flex-grow 按比例分配
- async/await 单线程并发 I/O
- PATCH vs PUT

### ❌ 2026-01-23 待加强
- ACID 的 D = **Durability**（不是 Duration）
- select_related vs prefetch_related
- N+1 查询问题

## 2026-01-22 测试记录

### ❌ GIL 对 I/O 密集型任务影响很小
- **错误认知**：以为 GIL 让多线程在 I/O 密集型任务中也无意义
- **正确理解**：
  - GIL 在等待 I/O 时会**释放**，其他线程可以运行
  - GIL 主要影响的是 **CPU 密集型**任务（纯计算）
  - I/O 密集型（网络请求、文件读写）→ 多线程仍然有效
  - CPU 密集型 → 用 `multiprocessing` 多进程

### ❌ ACID 事务属性（不知道）
| 字母 | 英文 | 中文 | 解释 |
|------|------|------|------|
| **A** | Atomicity | 原子性 | 事务要么全部成功，要么全部失败，不会"部分完成" |
| **C** | Consistency | 一致性 | 事务前后，数据库从一个有效状态到另一个有效状态 |
| **I** | Isolation | 隔离性 | 多个事务并发执行时，互不干扰 |
| **D** | Durability | 持久性 | 事务一旦提交，数据永久保存，断电也不丢 |

### 📝 幂等性 (Idempotent)（蒙对的，概念不清）
- **定义**：执行 1 次和执行 N 次，结果相同
- **HTTP 方法幂等性**：
  - `GET`：幂等（读取不改变状态）
  - `PUT`：幂等（用同样数据更新 100 次，结果还是那个状态）
  - `DELETE`：幂等（删除同一资源多次，结果都是"没了"）
  - `POST`：**不幂等**（每次可能创建新资源）
- **注意**：PUT 的标准定义是"用请求体完全替换资源"，不完全等同于 UPSERT

### ✅ Django ForeignKey on_delete=CASCADE
- 删除主表记录时，关联的从表记录也被删除
