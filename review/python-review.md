# Python 核心概念复习

## 2026-02-02 测试记录

### ✅ 已掌握
- **@dataclass frozen=True**: 不可变 → 可 hash → 可作 dict key。
- **装饰器 timer**: 完美实现，包含 @wraps。
- **list[str] vs List[str]**: 3.9+ 支持小写泛型。
- **with 语句**: __exit__ 清理资源。

### ❌ 待加强 / 新增盲点
- **[Core] __slots__**: 限制实例属性 + 节省内存（避免 __dict__ 开销）。
    ```python
    class Point:
        __slots__ = ['x', 'y']
    p = Point()
    p.z = 3  # ❌ AttributeError
    ```



## 2026-01-29 测试记录

### ✅ 已掌握
- **GIL (IO vs CPU)**: 彻底纠正了昨天的误区，理解 I/O 释放锁机制。
- **is vs ==**: 清楚 Identity (地址) vs Equality (值) 的区别。

### ❌ 待加强 / 新增盲点
- **[Core] Duck Typing**: (Pass) 核心是“关注行为而非类型”。"Walks like a duck..."。
- **[Core] Lambda 限制**: 只知道是匿名，忽略了 **"只能包含一个表达式 (Single Expression)"**，不能包含语句 (如 print, return, if-else 块)。
- **[Concept] Iterator vs Generator**:
    - **纠正**: Iterator 不一定耗费资源（它只是个指针），耗费资源的是 **Iterable List** (把所有数据存在内存里)。
    - Generator 是**惰性计算**的，省内存。

## 2026-01-28 测试记录

### ✅ 已掌握
- **@functools.wraps**: 之前是盲点，现在掌握了（保留元数据）。
- **Context Manager**: 熟练掌握 `__enter__` / `__exit__` 机制。
- **Args/Kwargs**: 理解元组 vs 字典的打包逻辑。

### ❌ 错题本 (Critical)
#### 1. GIL (Global Interpreter Lock) 🛑
- **用户误区**：认为 GIL 对 CPU 密集型无影响，对 I/O 有影响。
- **正确事实**：**完全相反**。
    - **CPU 密集型** (计算)：GIL 导致多线程无法利用多核，性能**差**。
    - **I/O 密集型** (网络/文件)：GIL 会在 I/O 等待时**释放**，多线程**有效**。

#### 2. Generator vs Iterator
- **关系**：Generator **是** 一种 Iterator。
- **区别**：Iterator 是实现了 `__next__` 的对象；Generator 是用 `yield` 语法糖自动实现的 Iterator（懒加载、状态挂起）。

#### 3. subprocess 安全
- 永远不要用 `string` 拼接命令。
- 使用 `list`：`['ls', '-l', filename]`。

## 2026-01-27 测试记录

### ✅ 已掌握
- **Context Manager**: `__enter__` & `__exit__`.
- **Type Hints**: `Union`.

### ❌ 待加强 / 新增盲点
- **[Decorator] @wraps**: 忘记了 `functools.wraps` 的作用（保留原函数元数据）。


## 2026-01-25 测试记录

### ✅ 已掌握
- **Shallow Copy**：准确预测了嵌套列表的变化。
- **Generator**：理解 yield 不会立即执行。
- **v-model ref**：JS 中必须加 .value。

### ❌ 待加强
- **Throttle vs Debounce**：例子记反了。
- **Pathlib parts**：未复习。


## 2026-01-24 测试记录

### ✅ 已掌握
- **Tuple vs List**：Tuple 不可变，List 可变。
- **Dataclass List陷阱**：理解 `[]` 会导致所有实例共享引用，需用 `field(default_factory=list)`。
- **Ref 解包**：Vue `ref` 在 Template 中自动解包，JS 中需 `.value`。

### ❌ 错题本
#### 1. `*args` vs `**kwargs`
- 核心区别：打包成的数据结构不同。
- `*args` (Arguments) -> **Tuple** (元组)。接收位置参数。
- `**kwargs` (Keyword Arguments) -> **Dict** (字典)。接收关键字参数。

#### 2. 浅拷贝 (Shallow Copy) 陷阱
- **场景**：`b = a.copy()` 或 `b = a[:]`。
- **现象**：修改 `b` 最外层元素，`a` 不变；但修改 `b` **内部嵌套的可变对象**（如 `b[0][0]=999`），`a` **也会变**！
- **原因**：浅拷贝只复制了“容器”，里面的元素引用的还是原来的对象。
- **解法**：`import copy; b = copy.deepcopy(a)`。

#### 3. Generator (生成器)
- 带有 `yield` 的函数调用时返回 **Generator 对象**。
- 不会立即执行，需 `next()` 或 `for` 循环触发。
