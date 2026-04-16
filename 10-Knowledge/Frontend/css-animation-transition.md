# 前端 CSS 动画与过渡 (Frontend Animation & Transition)

> [!NOTE]
> 收录前端开发中关于 CSS 动画、Vue Transition 组件及动效设计的核心知识点和最佳实践。

---

## 1. Vue Transition Modes

在 Vue 的 `<Transition>` 组件中，`mode` 属性决定了元素进入 (Enter) 和离开 (Leave) 的时序，这对于列表更新或路由切换的视觉体验至关重要。

### 1.1 模式对比

| 模式 | 描述 | 行为特征 | 适用场景 |
|------|------|----------|----------|
| **Default** (无) | 同时进行 | 新元素进入和旧元素离开**同时发生**。两者的 DOM 在某一时刻会共存。 | 需要两个元素重叠的特效 (如 Cross-fade)。通常需要绝对定位防止布局崩塌。 |
| **`out-in`** | **先出后进** | 旧元素先执行离开动画，**完全消失后**，新元素才开始执行进入动画。 | **卡片切换、路由跳转**、标签页切换。能确保布局平滑，无重叠挤压。 |
| **`in-out`** | 先进后出 | 新元素先进入，动画完成后，旧元素才离开。 | 较少使用，特殊视觉需求（如新图层覆盖旧图层）。 |

### 1.2 为什么推荐 `out-in`？

在大多数**内容替换**（Replace）的场景下，默认模式会导致布局跳动：
1. 旧元素还没消失，新元素已经插入 DOM。
2. 容器高度瞬间被撑大（包含两个元素的高度）。
3. 旧元素消失，容器高度回缩。

使用 `mode="out-in"` 可以完美规避这个问题，让用户感觉内容是“平滑替换”的。

### 1.3 代码示例

```vue
<template>
  <div class="card-container">
    <Transition name="fade" mode="out-in">
      <div :key="currentIndex" class="card">
        {{ currentContent }}
      </div>
    </Transition>
  </div>
</template>

<style>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

---

*创建日期: 2026-01-30*
