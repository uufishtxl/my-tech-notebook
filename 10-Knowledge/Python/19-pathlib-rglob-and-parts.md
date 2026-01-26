# Python Pathlib: rglob 与 parts 详解

> [!NOTE]
> `pathlib` 是 Python 3.4+ 引入的面向对象路径处理库。相比旧的 `os.path` 字符串操作，它更安全、更直观，尤其在处理复杂的文件系统树时。

---

## 1. rglob (Recursive Glob) —— 递归搜索

### 是什么
`rglob` 是 `Path` 对象的一个方法，用于递归地匹配文件模式。它等同于在 `glob` 模式前加上了 `**/`。

### 代码示例
```python
# 找到当前目录下及其所有子目录中的 DITA 文件
for file in path.rglob('*.dita'):
    print(file) # 返回的是 Path 对象
```

### 关键点
- **深度优先**：它会遍历所有子文件夹。
- **返回生成器**：它返回的是一个 `Generator`，在大规模文件系统下不会一次性占用大量内存。

---

## 2. parts 属性 —— 路径拆解

### 是什么
`parts` 返回一个元组，包含路径的各个组件（盘符、文件夹名、文件名）。

### 为什么用它（相比字符串比对）
传统的字符串检查 `if "temp" in str(path)` 可能会误伤。使用 `parts` 可以精确匹配路径层级。

### 代码演示
```python
p = Path('/etc/nginx/conf.d/default.conf')
print(p.parts) 
# 输出: ('/', 'etc', 'nginx', 'conf.d', 'default.conf')

# 检查是否在某个特定文件夹下（无视层级）
if 'temp' in p.parts:
    print("跳过临时文件")
```

---

## 3. 这里的工程实践

在 `dita_parser.py` 中，我们结合了这两个特性：

```python
# 1. 递归扫描所有 DITA
for dita_file in self.dita_root.rglob('*.dita'):
    # 2. 只有当路径的任何一级文件夹名为 'temp' 或 'out' 时，才过滤掉
    # 这种写法比字符串搜索 str(path) 更清晰、更稳健
    if 'temp' in dita_file.parts or 'out' in dita_file.parts:
        continue
```

---

## 注意事项
- `rglob` 不会跟踪符号链接（Symlinks），除非在某些特定配置下。
- `parts` 返回的是不可变的 `tuple`，你可以安全地用 `in` 进行数组成员检查。

---

*创建日期: 2026-01-24*
