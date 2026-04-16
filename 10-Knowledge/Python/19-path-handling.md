
## 路径处理

`os.path` 处理后缀名简直反人类，需要切片。`pathlib` 是直接访问属性。

假设路径是：`p = Path('/usr/bin/python3.8.tar.gz')`

|**需求**|**老派 (os.path)**|**新派 (pathlib)**|**备注**|
|---|---|---|---|
|**文件名**|`os.path.basename(p)`|**`p.name`**|`'python3.8.tar.gz'`|
|**主文件名**|`os.path.splitext(os.path.basename(p))[0]` 🤮|**`p.stem`**|`'python3.8.tar'` (注意是最后一个点之前)|
|**后缀名**|`os.path.splitext(p)[1]`|**`p.suffix`**|`'.gz'`|
|**父目录**|`os.path.dirname(p)`|**`p.parent`**|返回父级 Path 对象|

### 3. 创建文件夹 (Create Directory)

创建多级目录时，`pathlib` 的语义更清晰。

- **老派 (`os`)**：    
    ```Python
    if not os.path.exists('data/logs'):
        os.makedirs('data/logs') 
    ```
    
- **新派 (`pathlib`)**：
    ```python
    p = Path('data/logs')
    # parents=True: 自动创建缺失的父目录 (相当于 mkdir -p)
    # exist_ok=True: 如果文件夹已存在，不报错
    p.mkdir(parents=True, exist_ok=True) 
    ```
    

### 4. 读写文件 (Read/Write)

对于简单的文本读写，`pathlib` 可以让你少写 `with open(...)` 缩进块。

- **老派 (标准写法)**：
    ```python
    with open('settings.py', 'r', encoding='utf-8') as f:
        content = f.read()
    ```
    
- **新派 (`pathlib` 快捷方式)**：
    ```python
    # 一行搞定读取
    content = Path('settings.py').read_text(encoding='utf-8')
    
    # 一行搞定写入
    Path('log.txt').write_text("Error occurred!")
    ```
### 5. 查找文件 (Globbing)

`pathlib` 把 `glob` 模块的功能也集成进来了。

- **老派 (`glob` 模块)**：
    ```python
    import glob
    files = glob.glob('src/**/*.py', recursive=True)
    ```
    
- **新派 (`pathlib`)**：
    ```python
    p = Path('src')
    # rglob = Recursive Glob (递归查找)
    files = list(p.rglob('*.py')) 
    ```
## Python Pathlib: rglob 与 parts 详解

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


## 4. 特殊陷阱：Path 实例与字符串

许多第三方库（如 `chromadb.PersistentClient` 或 `os.listdir`）接受路径参数，但它们往往不接受 `.name`。

-   **Path 对象本身**：代表了完整的文件/文件夹路径（可能是相对的也可能是绝对的）。
-   **`.name`**：只代表并返回了路径中的**最后一部分（基本名称）**。

### 错误案例
```python
# p 是 Path("/app/data/chroma_db")
client = chromadb.PersistentClient(path=p.name) 
# 这会把 path 设为 "chroma_db"，导致数据库在**当前目录下**创建（而不是 /app/data 下）
```

### 正确案例
保留完整路径信息，直接传递 Path 对象（如果库支持）或者转为字符串：

```python
# 方案 A (推荐)：直接传 Path 对象 (许多现代库如 Pandas, Transformers 都支持)
client = chromadb.PersistentClient(path=persist_directory)

# 方案 B (最稳妥)：显式转为字符串
client = chromadb.PersistentClient(path=str(persist_directory))
```

> [!IMPORTANT]
> 永远不要为了得到一个字符串而使用 `.name`，除非你明确只想拿到文件名/最后一级目录名。如果要拿到路径的字符串形式，请使用 `str(p)`。
