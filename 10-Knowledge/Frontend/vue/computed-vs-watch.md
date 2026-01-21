# Vue computed 与 watch 的使用边界

#vue #computed #watch #async

## 核心原则

| API | 用途 | 支持异步 |
|-----|------|----------|
| `computed` | 同步派生状态 | ❌ |
| `watch` | 响应变化执行副作用 | ✅ |

## computed 的限制

```typescript
// ❌ 反模式：computed 中使用 async
const title = computed(async () => {
  const response = await api.get(`/audios/${chunk.value.source_audio}`)
  return response.data.title  // 返回的是 Promise，不是值！
})
```

**问题**：
- `computed` 返回 `Promise` 对象
- 模板显示 `[object Promise]`

## 正确做法：watch

```typescript
const title = ref<string | null>(null)

watch(chunk, async (newChunk) => {
  if (newChunk?.source_audio) {
    const response = await api.get(`/audios/${newChunk.source_audio}`)
    title.value = response.data.title
  }
})
```

> [!IMPORTANT]
> **`computed` 用于同步派生，`watch` 用于异步副作用。**

## watch 选项

```typescript
watch(source, callback, {
  immediate: true,  // 初始化时立即执行
  deep: true        // 深度监听对象/数组内部变化
})
```

> [!WARNING]
> `deep: true` 有性能开销，大型数据结构慎用。
