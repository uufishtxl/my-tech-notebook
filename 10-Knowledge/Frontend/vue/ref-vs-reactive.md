# Vue 3 响应式 API: ref vs reactive

#vue #reactivity #composition-api

## 核心区别

| 特性 | `ref` | `reactive` |
|------|-------|------------|
| 数据类型 | 任意类型 | 仅对象/数组 |
| 访问方式 | 需要 `.value` | 直接访问属性 |
| 重新赋值 | ✅ 支持 | ❌ 丢失响应性 |

## ref - 万能"盒子"

```typescript
const count = ref(0)
const user = ref({ name: 'Edith' })

// 访问需要 .value
count.value++
user.value = { name: 'New' }  // ✅ 可以重新赋值
```

**适用场景**：
- 基本类型（必须用 ref）
- 需要完整替换的对象/数组

## reactive - 对象"解包"

```typescript
const selection = reactive({
  dramaId: null,
  season: null,
  episode: null
})

// 直接访问，无需 .value
selection.dramaId = 123
```

**适用场景**：
- 一组相关属性的对象
- 不需要整体替换的状态

## 决策建议

> **优先使用 `ref`。当有多个相关属性需要作为整体管理时，再考虑 `reactive`。**

## 面试要点

这是高频 Vue 3 面试题，考察：
1. 响应式系统理解
2. `.value` 存在的原因
3. `reactive` 解构丢失响应性问题（用 `toRefs` 解决）

---

*相关：[[21-ref-vs-let|ref vs let 何时用响应式]]*
