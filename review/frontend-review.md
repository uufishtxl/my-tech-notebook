# Frontend & Network 复习笔记

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
