# Python `with` 语句与上下文管理器

#python #context-manager

## 基本语法

```python
with open('file.txt', 'r') as f:
    content = f.read()
# 离开 with 块后，文件自动关闭
```

## 工作原理

`with` 语句自动调用对象的 `__enter__` 和 `__exit__` 方法：

```python
class MyContext:
    def __enter__(self):
        print("进入")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("离开（无论是否异常）")

with MyContext() as ctx:
    print("执行中")
```

## 常见使用场景

| 场景 | 示例 |
|------|------|
| 文件操作 | `with open(...) as f:` |
| 数据库连接 | `with connection.cursor() as cursor:` |
| 锁操作 | `with threading.Lock():` |
| 临时目录 | `with tempfile.TemporaryDirectory() as tmpdir:` |

## tempfile.TemporaryDirectory

```python
import tempfile

with tempfile.TemporaryDirectory() as tmpdir:
    # tmpdir 是临时目录路径
    # 在 with 块内执行操作
    pass
# 离开后，临时目录及其内容自动删除
```

> [!TIP]
> 使用 `with` 可以确保资源（文件、连接、锁等）在使用后正确释放，即使发生异常。
