# Python `reversed()` Function

## 1. 核心概念
- **作用**: 返回一个**反向迭代器 (Reverse Iterator)**，用于倒序遍历序列。
- **返回值**: `<list_reverseiterator object>` (迭代器)，**不是列表**。
- **副作用**: **不修改原列表** (Non-destructive)。原列表保持原顺序。

```python
ml = [1, 2, 3]
rev_iter = reversed(ml)

print(rev_iter) 
# Output: <list_reverseiterator object at 0x...>

print(list(rev_iter)) 
# Output: [3, 2, 1]

print(ml) 
# Output: [1, 2, 3] (原列表不变)
```

## 2. 常见用法
可以直接在 `for` 循环中使用，因为 `for` 循环会自动处理迭代器：

```python
for i in reversed(ml):
    print(i)
# Output: 3, 2, 1
```

## 3. 对比 `list.reverse()`
- **`list.reverse()`**: **就地修改 (In-place)** 原列表，返回 `None`。
- **`reversed()`**: 不修改原列表，返回迭代器。

```python
ml = [1, 2, 3]
ml.reverse()
print(ml) # [3, 2, 1] (原列表已变)
```

## 4. 最佳实践
- **只读遍历**: 用 `reversed()`。
- **需要反向列表副本**: 用 `list(reversed(ml))` 或切片 `ml[::-1]`。
- **必须修改原列表**: 用 `ml.reverse()`。
