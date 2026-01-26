# Python 核心概念复习

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
