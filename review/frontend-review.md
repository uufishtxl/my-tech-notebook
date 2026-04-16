# Frontend & Network 复习笔记

## 2026-02-02 测试记录

### ✅ 已掌握
- **Vue computed 缓存**: 核心理解正确（依赖变化才重算）。
- **watch vs watchEffect**: 区分清晰。
- **Flexbox gap**: 现代写法掌握。
- **HTTP 304**: Not Modified，利用缓存。

### ❌ 待加强 / 新增盲点
- **[TS] Pick<T, K>**: 从 T 中选取 K 指定的属性。
    ```typescript
    type UserBasic = Pick<User, 'id' | 'name'>
    ```
    相关：`Omit<T, K>` 是反向操作（排除属性）。



## 2026-01-29 测试记录

### ✅ 已掌握
- **Boolean(null)**: 纠正成功 (False)。
- **BFC**: 理解正确 (布局隔离 / 解决 Margin 塌陷)。
- **Tailwind Shadow**: 正确 (box-shadow)。

### ❌ 待加强 / 新增盲点
- **[Vue] Ref vs Reactive**:
    - 官方推荐：**优先使用 `ref`** (即便对对象)。
    - 原因：`reactive` 解包后丢失响应性，且替换整个对象时会断连。`ref` 更加一致 (`.value`)。
- **[Media] Events**: `loadedmetadata` (有时长了) -> `canplay` (缓冲够了)。

## 2026-01-28 测试记录

### ✅ 已掌握
- **CORS Preflight**: 掌握了 OPTIONS 请求及触发条件。
- **v-if vs v-show**: 性能与渲染机制理解正确。
- **computed vs watch**: 适用场景区分清晰。

### ❌ 待加强 / 新增盲点
- **[JS] Null vs Undefined Boolean**: 错误认为 `Boolean(null)` 是 true。实际上两者转 Boolean **都是 false**。
- **[CSS] BFC**: 盲点。Block Formatting Context 是独立的渲染区域（解决 margin 重叠、清除浮动）。

## 2026-01-27 测试记录

### ✅ 已掌握
- **Vue Props/Emit**: 数据流向清晰。
- **Vue Lifecycle**: `onMounted` 最佳实践。
- **Flexbox**: `flex-grow` 掌握。

### ❌ 待加强 / 新增盲点
- **[Network] CORS Preflight**: 不知道 OPTIONS 请求的作用。
- **[JS] Null vs Undefined**: 误认为 `Boolean(null)` 是 true。


## 2026-01-24 测试记录

### ✅ 已掌握
- **v-if vs v-show**：销毁 DOM vs `display: none`。
- **Flex Wrap**：控制换行。
- **Ref .value**：JS 中必须用 `.value`。

### ❌ 错题本
#### 1. TypeScript Partial<T>
- **作用**：将 T 中所有属性变为 **可选 (Optional, ?)**。
- 比如 `User { name: string }` -> `Partial<User>` 变成 `{ name?: string }`。

#### 2. Connection vs Read Timeout
- **Connection Timeout**：**打通电话之前**。连接建立超时（如服务器挂了、防火墙屏蔽）。
- **Read Timeout**：**打通电话之后**。连接已建立，正在等待服务器吐数据（如服务器处理太慢、AI 生成太久）。
- **应用**：我们给 AI 分析加的 60s/90s 是 **Read Timeout**。
