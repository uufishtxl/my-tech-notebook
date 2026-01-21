# JavaScript 数组与字符串常用方法

#javascript #array #string

## 数组拷贝：浅拷贝 vs 深拷贝

### 浅拷贝 - `[...arr]`

```javascript
const copy = [...original]  // 新数组，但元素是引用
copy[0].name = 'changed'    // ⚠️ 原数组也被修改
```

### 深拷贝 - `JSON.parse(JSON.stringify())`

```javascript
const copy = JSON.parse(JSON.stringify(original))
copy[0].name = 'changed'    // ✅ 原数组不受影响
```

> [!TIP]
> 需要完全隔离的副本时用深拷贝，如保存原始数据用于对比。

---

## 字符串排序：`localeCompare()`

```javascript
dramas.sort((a, b) => a.name.localeCompare(b.name))
```

**为什么不用 `<` / `>`？**
- `<` 基于 Unicode 编码，非英文字符排序不符合直觉
- `localeCompare()` 根据语言规则"自然排序"

---

## 数组检查：`some()`

```javascript
// 检查是否存在满足条件的元素
const exists = serverDramas.some(d => d.id === newDramaId)
```

**特点**：找到后立即停止（短路），高效。

---

## TypeScript Map 泛型

```typescript
const map = new Map<string, RegionInfo>()
//              ↑ 键类型   ↑ 值类型
```

**Map vs Object**：
- Map 支持任意键类型
- Map 保证插入顺序
- 频繁增删用 Map 更快
