# Python *args 和 **kwargs 详解

## 核心区别
| 写法 | 名称 | 数据类型 | 作用 |
| :--- | :--- | :--- | :--- |
| `*args` | Positional Arguments | **Tuple** (元组) | 接收多余的**位置参数** |
| `**kwargs` | Keyword Arguments | **Dict** (字典) | 接收多余的**关键字参数** |

## 代码演示

### 1. 基础用法
```python
def demo(a, *args, **kwargs):
    print(f"a: {a}")
    print(f"args: {args} (Type: {type(args)})")
    print(f"kwargs: {kwargs} (Type: {type(kwargs)})")

# 调用
demo(1, 2, 3, x=4, y=5)

# 输出：
# a: 1
# args: (2, 3) (Type: <class 'tuple'>)
# kwargs: {'x': 4, 'y': 5} (Type: <class 'dict'>)
```

### 2. 解包 (Unpacking)
当我们调用函数时，也可以用 `*` 和 `**` 来**拆开**列表或字典传进去。

```python
def add(x, y, z):
    return x + y + z

nums = [1, 2, 3]
# add(nums) -> 报错，因为只传了1个参数（整个列表）
print(add(*nums)) # -> 6 (等价于 add(1, 2, 3))

params = {'x': 10, 'y': 20, 'z': 30}
print(add(**params)) # -> 60 (等价于 add(x=10, y=20, z=30))
```

## 常见场景
1. **装饰器 (Decorators)**：不仅要包装函数，还要原封不动地传递参数。
   ```python
   def my_decorator(func):
       def wrapper(*args, **kwargs):
           print("Before call")
           return func(*args, **kwargs) # 原样透传
       return wrapper
   ```
2. **类继承 (Subclassing)**：调用父类 `__init__` 时。
   ```python
   class MyView(BaseView):
       def __init__(self, *args, **kwargs):
           super().__init__(*args, **kwargs)
           self.custom_attr = "hello"
   ```
