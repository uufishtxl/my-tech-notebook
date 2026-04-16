# Python Dictionary Update and Merge

## 1. `update()` Method
- **语法**: `dict.update(other_dict)`
- **作用**: **就地更新 (In-place update)**。将 `other_dict` 的键值对合并到当前字典中。
- **返回值**: `None`。这是 Python 的设计哲学，表示该操作修改了对象本身，而不是返回新对象。

```python
d = {"a": 1}
print(d.update({"b": 2}))  # Output: None
print(d)                   # Output: {'a': 1, 'b': 2}
```

## 2. 合并操作符 (Python 3.9+)

### `|=` (In-place Update Operator)
- **等价于**: `update()`
- **作用**: 就地修改左侧字典。
- **返回值**: `None` (作为表达式单独使用)

```python
d = {"a": 1}
d |= {"b": 2}  # d 变为 {'a': 1, 'b': 2}
```

### `|` (Merge Operator)
- **作用**: **创建新字典**。不修改原字典，返回一个新的合并后的字典。
- **返回值**: 新的字典对象。

```python
d1 = {"a": 1}
d2 = {"b": 2}
d3 = d1 | d2
print(d3) # {'a': 1, 'b': 2}
print(d1) # {'a': 1} (Unchanged)
```

## 3. 核心区别
- **就地修改 (In-place)**: `dict.update()`, `|=` -> 返回 `None`，改动原对象。
- **创建新对象 (New Object)**: `|` -> 返回新字典，原对象不变。
