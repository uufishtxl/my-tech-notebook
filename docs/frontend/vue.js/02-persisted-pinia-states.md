# 持久化 Pinia 状态

在过去，比如使用 `Vuex` 进行状态管理时，我们常需要手动将 Token 之类的值存入 `localStorage`，并在应用加载时手动读出，以防页面刷新时丢失数据。

在使用 Pinia 的项目中，我们可以有更优雅的管理方式。

Pinia 本身默认**只在内存中**管理状态，并不会进行持久化。为了解决刷新丢失数据的问题，我们需要一个“两步走”的策略：

1. **安装并注册插件**： 我们首先需要安装 `pinia-plugin-persistedstate` 插件。然后，在 `main.ts` 文件中，我们必须在创建 Pinia 实例后，通过 `pinia.use(piniaPluginPersistedstate)` 来注册这个插件。

2. **在 Store 中开启持久化**： 完成注册后，我们就可以在 `defineStore` 时，通过添加 `{ persist: true }` 这个配置项，来“告诉”插件：“请帮我持久化这个 Store 里的所有状态。

```TypeScript
// --- 1. 基础导入 ---
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import './styles/styles.css'

// --- 2. 插件和 Store 导入 (新增) ---
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'
import axios from 'axios'
import { useAuthStore } from './stores/authStore' 

// --- 3. Pinia 实例创建与配置 (新增) ---
const pinia = createPinia()
pinia.use(piniaPluginPersistedstate) // 需要使用持久状态机制

// --- 4. Vue App 实例创建与配置 ---
const app = createApp(App)
app.use(pinia) // <--- 【使用配置好的 pinia

// --- 5. 状态恢复逻辑 (新增) ---
// ！！注意：这必须在 app.use(pinia) 之后 ！！
try {
  const authStore = useAuthStore() // 1. 获取 store 实例

  // 2. 检查 store 中是否已恢复了 token
  if (authStore.accessToken) {
    // 3. 手动恢复 Axios 的全局 Header
    axios.defaults.headers.common['Authorization'] = `Bearer ${authStore.accessToken}`
  }
} catch (error) {
  console.error("Failed to re-initialize auth header:", error)
}

// --- 6. 挂载 App ---
app.use(router)
app.mount('#app')
```

[关于这个知识点的面试题](../../../interviews/vue/01-application-status-and-life-cycle.md)