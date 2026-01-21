# Python > `with`语句

`with`语句是 Python 中非常重要且强大的语法。其背后的原理是“上下文管理”。

在编程中，我们经常会使用一些需要“打开”和“关闭”的资源，比如：
* 文件（打开后必须关闭）
* 网络连接（建立后必须断开）
* 数据库事务（开始后必须提交或回滚）
* 临时目录（创建后必须删除）

如果手动管理这些资源，通常会写出这样的代码：

```Python
f = open('my_file.txt', 'w')
try:
	f.write('hello')
	# 这里可能会发生错误
finally:
	# 无论 try 块是否发生错误，finally 块中的代码都一定被执行
	f.close()
	print('文件已关闭')
```

`try...finally...`这种模式虽然能够保证资源被关闭，但它很繁琐，而且容易忘记写 `finally` 块，从而导致资源泄露。

`with` 语法就是为了简化 `try...finally...`这种模式而生的。它提供了一种更优雅、更安全的方式来管理资源。

```Python
with open('my_file.txt', 'w') as f:
	f.write('hello')
	# 这里可能发生错误
# 一旦 with 代码块结束，Python 会自动调用 f.close()
print('文件已关闭')
```

* `with...as...`：开启一个上下文管理代码块。
* 自动清理：无论 `with` 代码块是否正常执行完毕，还是中途因为错误而中断，Python 都会保证在退出这个代码块时，执行必要的清理操作（比如关闭文件）。

## 背后的逻辑

任何能够被 `with` 语句使用的对象，都必须遵守“上下文管理协议”，也就是它的类必须实现两个特殊方法：
* `__enter__(self)`:
	* 在进入 `with` 代码块时被调用
	* 它返回的值会被赋给 `as` 后面的变量（比如 `f`）
* `__exit__(self)`:
	* 在退出 `with`代码块时被调用（无论何种原因退出）
	* 所有的清理逻辑都写在这个方法里
	* 如果 `with`块发生了异常，异常的类型、值和追溯信息回座位参数传给 `__exit__`，让它可以根据情况进行处理。
## 我们用到的两个上下文对象情景

### 打开/关闭文件

```Python
# 'r' for 'read', 'b' for 'binary', means reading a binary file
with open(chunk_path, 'rb') as f:
	# ...
```

### 创建/清除临时目录

```Python
with tempfile.TemporaryDirectory() as temp_dir:
	# ...
```
- `TemporyDirectory()` 返回的也是一个上下文管理器对象。
- 它的 `__enter__()`方法负责在磁盘上创建一个唯一的临时目录，并返回这个目录的路径字符串，所有 `temp_dir` 就是这个路径
- 它的 `__exit__` 方法负责递归地删除整个临时目录及其所有内容。
