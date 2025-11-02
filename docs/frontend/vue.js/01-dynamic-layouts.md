# 动态布局

与 Quasar / Nuxt 这类元框架会内置对动态布局的支持。在纯净的 `Vite + Vue Router`项目中，我们也可以用极其优雅的方式来完美复刻这样的特性。

**实现方式**：根据当前路由（Route），动态地为页面（`<RouterView />`）包裹上一个正确的父布局组件（例如`AuthLayout`或`AppLayout`）。

## 步骤一：创建“布局”组件（Layouts）

* 可以在 `src/layouts/` 文件夹中创建布局组件。
* **关键点**：每一个布局组件都必须包含一个 `<slot />`插槽。将路由页面渲染到插槽中。

## 步骤二：在路由器中标记路由

告诉 `vue-router` 每一个路由偏好哪种布局。我们使用 `meta` 字段来配置。

```Typescript
const router = createRouter({
    # ... 
    routers: [
        {
            path: '/',
            name: 'home',
            component: Home,
            meta: {
                layout: 'AppLayout' // 使用 App 布局
            }
        }
    ]
})
```

## 步骤三：在 `App.vue` 中实现“动态切换器”（用 `is component`）

`App.vue`会检查当前的路由标签，从而穿上正确的“皮肤”，也就是指定使用哪一种布局。

```HTML
<template>
  <component :is="layoutComponent">
    <RouterView /> 
  </component>
</template>
```

```TypeScript
<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { RouterView } from 'vue-router' // 别忘了导入 RouterView

// 1. 导入我们所有的布局组件
import AppLayout from '@/layouts/AppLayout.vue'
import AuthLayout from '@/layouts/AuthLayout.vue'

// 2. 获取当前路由信息 (这是响应式的)
const route = useRoute()

// 3. (可选, 但很规范) 定义一个类型安全的布局映射
//    这能解决我们上次遇到的 TypeScript 报错
type LayoutMap = {
  [key: string]: typeof AppLayout | typeof AuthLayout
}
const layouts: LayoutMap = {
  AppLayout: AppLayout,
  AuthLayout: AuthLayout,
}

// 4. 【核心逻辑】
//    创建一个“计算属性”，它会*自动*跟踪路由变化
const layoutComponent = computed(() => {
  // 5. 检查路由的 'meta.layout' 标签
  const layoutName = route.meta.layout as string | undefined
  
  // 6. 如果标签存在, 并且在我们的映射表里, 就返回它
  if (layoutName && layouts[layoutName]) {
    return layouts[layoutName]
  }
  
  // 7. (兜底) 如果路由没有标记, 或标记错误,
  //    默认返回“App”布局 (这是一个安全的选择)
  return AppLayout
})
</script>
```




