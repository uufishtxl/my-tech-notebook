# 路由守卫

## 什么是路由守卫

顾名思义，路由守卫就像是 Vue Router 的保安。每当用户视图从一个路由（页面）跳转到另一个路由时，它都会介入，执行一系列检查。并根据检查结果，做出三个决定：

* **放行（`next()`）**：允许用户访问他们想访问的页面；
* **重定向（`next({name: 'login'})`）**：把用户“踢”到另一个页面（比如登录页）；
* **取消（`next(false)`）**：阻止这次跳转，让用户停留在当前页面。

## 守卫的类型

主要有三类，但是 90% 的情况只会用到**全局守卫**和**组件内守卫**。

* **全局守卫**
  * `router.beforeEach`：全局前置守卫。每次路由切换之前都会被调用。这是实现认证（`requireAuth`）的完美地点。
  * `router.afterEach`：全局后置守卫。每次路由切换之后被调用。它无法阻止导航，通常用于**修改页面标题**（`document.title`）或发送页面浏览分析。

* **组件内守卫**
  * `onBeforeRouteLeave`：当用户试图离开当前组件时被调用
  * `onBeforeRouteUpdate`: 当路由概念，但组件被复用时调用（例如：从`/user/1`导航到`/user/2`）
  * `beforeRouteEnter`：在导航进入组件之前调用

* **路由独享守卫**
  * `beforeEnter`：直接在路由配置对象（`router/index.ts`）中定义，只对那一个路由生效。

## 四个常见的守卫场景

* 防止未登录用户访问
* 防止已登录用户访问
* 防止用户在未有保存更改时离开
  * 守卫类型：组件内守卫：`onBeforeRouteLeave`
    ```HTML
    <script setup lang="ts">
    import { onBeforeRouteLeave } from 'vue-router'
    import { ref } from 'vue'

    const isFormDirty = ref(false) // 假设表单内容变化时，这个值会变成 true

    onBeforeRouteLeave((to, from, next) => {
    if (isFormDirty.value) {
        // 表单脏了，弹窗确认
        const answer = window.confirm('您有未保存的修改，确定要离开吗？');
        if (answer) {
        next(); // 确认离开，放行
        } else {
        next(false); // 取消离开，留在当前页
        }
    } else {
        // 表单是干净的，直接放行
        next();
    }
    });
    </script>
    ```
* **基于角色的访问**
  * 结合 Meta 标签与 `beforeEach` 扩展逻辑实现。
    ```TypeScript
    router.beforeEach((to, from, next) => {
    const authStore = useAuthStore();
    const targetRoles = to.meta.roles as string[] | undefined; // 获取路由需要的角色

    if (to.meta.requiresAuth && !authStore.isAuthenticated) {
        // 1. 未登录
        next({ name: 'login' });
    } else if (targetRoles && !targetRoles.includes(authStore.user.role)) {
        // 2. 登录了，但角色不匹配
        // (例如：用户是 'user'，但页面需要 'admin')
        next({ name: '403-forbidden' }); // 踢到“禁止访问”页
    } else {
        // 3. 放行
        next();
    }
    });
    ```
* 特定状态认证：在特定页面跳转路由时携带凭证，然后在`beforeEach`中检查是否携带此凭证，比如注册账户后的邮箱验证页面。（更高级的场景为基于用户特定状态的认证）。
  * Meta 标签：`meta: { requiresUnverified: true }`
  * Store 状态：authStore 必须提供 isVerified 状态。
  * 全局守卫：在 beforeEach 中补充逻辑。
  * 逻辑
    ```JavaScript
    router.beforeEach((to, from, next) => {
    const authStore = useAuthStore();

    // ... (场景一 和 场景二 的代码) ...

    // 场景五：特定状态 (新)
    } else if (to.meta.requiresUnverified && authStore.isAuthenticated && authStore.isVerified) {
        // 访问“未验证”页，但用户“已验证”
        next({ name: 'home' }); // 踢回首页

    } else {
        // 其他所有情况，放行
        next();
    }
    });
    ```


## 关键陷阱与注意事项
* `next()` 必须且只能被调用一次：在任何 `beforeEach` 检查中，`next()` 必须被调用一次。不调用，导航会被卡住；调用多次，会报错。
* **F5 刷新问题**：
  * 当用户按 F5 刷新页面时，`beforeEach` 依然会执行。
  * 此时，`from` 路由是无效的（`from.name` 是 `null`）。
  这就是为什么你的守卫逻辑永远不要依赖 `from`，而必须只依赖 `authStore`（它会从 `localStorage` 自动恢复状态）。
* **异步守卫**：`beforeEach` 可以是一个 `async` 函数。如果你在守卫中需要 `await` 一个 `API` 请求（比如检查 `Token` 有效性），这是可以的。但一定要在 `await` 之后再调用 `next()`。