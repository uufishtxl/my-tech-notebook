### 1. 什么是装饰器？为什么要用它？

- **什么是装饰器 (Definition)**： 它是一种**“在不修改原函数代码的前提下，给函数增加新功能”**的设计模式。 它本质上是一个**高阶函数**（接收一个函数，返回一个新的函数）。
    
- **为什么要用 (Why)**： 为了 **DRY (Don't Repeat Yourself)**。 当你发现有好几个函数都需要“登录检查”、“记录日志”、“计算耗时”时，不要把这些重复代码写进每个函数里，而是把它们提取出来做成装饰器，像贴纸一样贴上去。
    

---

### 2. 怎么称呼它们？

Python

```
@timer             # <--- 1. 装饰器 (Decorator)
def add(a, b):     # <--- 2. 原函数 / 被装饰函数 (Decorated Function / Original Function)
    pass
```

- **语法糖 (Syntactic Sugar)**：`@timer` 只是 Python 给你的特权。它等价于写了 `add = timer(add)`。
    

---

### 3. 装饰器的标准结构

记住这个**“三层汉堡”**结构：

```python
from functools import wraps

def 装饰器名(func):                  # Layer 1: 工厂层 (接收原函数)
    
    @wraps(func)                    # <--- 贴假身份证
    def wrapper(*args, **kwargs):   # Layer 2: 包装层 (接收任意参数)
        
        # ... A. 也就是执行前的逻辑 (Before) ...
        
        result = func(*args, **kwargs)  # <--- B. 核心：执行原函数 (透传参数)
        
        # ... C. 也就是执行后的逻辑 (After) ...
        
        return result               # <--- D. 归还结果
        
    return wrapper                  # Layer 3: 返回包装后的新函数
```

---

### 4. Wrapper 在做什么？要传什么？

- **Wrapper 的身份**：它是**替身**。原函数 `add` 其实已经被藏起来了，现在在外面干活的其实是 `wrapper`。
    
- **为什么要传 `*args` 和 `**kwargs`**： 为了做一个**万能适配器**。 因为装饰器开发者不知道用户会把它用在无参函数 `func()` 上，还是复杂参数 `func(a, b=1)` 上。只有同时写上这两个，才能保证**“不管用户传什么，我都能接住并原封不动地传给原函数”**。
    

---

### 5. @wraps 和元数据

- **元数据 (Metadata)**：指函数的**“身份证信息”**，主要是：
    
    - `__name__` (函数名)
        
    - `__doc__` (文档注释)
        
- **为什么要用 `@wraps(func)`**： 如果不加，`add` 函数的名字就会变成 `wrapper`，注释也会丢失。这会导致调试困难（报错报在 wrapper 里），API 文档生成失败。 `@wraps` 的作用就是把原函数 `func` 的身份证**复印一份**，贴在替身 `wrapper` 的脑门上，骗过所有人。
    
- **如何查看元数据**：

    ```python
    print(add.__name__)  # 查看名字
    print(add.__doc__)   # 查看注释
    help(add)            # 查看详细信息 (如果没有 wraps，这里会显示 wrapper 的信息)
    ```
---

### 6. Return Result (接力棒)

- **核心逻辑**： `wrapper` 既然是替身，它就必须模仿得天衣无缝。 老板（调用者）是要结果的（比如 `add` 算的 `30`）。原函数算出来了，`wrapper` 拿到手后，必须**return 出去**。
    
- **如果不写 return**： Python 函数默认返回 `None`。那你辛苦计算的结果就被 `wrapper` 私吞了，调用者只能收到 `None`，程序逻辑就会断裂。