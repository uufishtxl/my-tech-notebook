# Vue Router 参数获取

#vue #vue-router

## 动态路由配置

```typescript
// router/index.ts
{
  path: '/slicer/workbench/:id',  // :id 为动态参数
  name: 'audio-workbench',
  component: AudioWorkbench
}
```

## 组件内获取参数

```typescript
import { useRoute } from 'vue-router'

const route = useRoute()

onMounted(() => {
  const chunkId = route.params.id
  // 访问 /slicer/workbench/123 时，chunkId = '123'
})
```

## 常用属性

| 属性 | 说明 |
|------|------|
| `route.params` | 路径参数 (`:id`) |
| `route.query` | 查询参数 (`?foo=bar`) |
| `route.name` | 路由名称 |
| `route.path` | 当前路径 |
