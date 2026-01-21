# Tailwind CSS Grid 响应式布局

#tailwind #css #grid

## 自适应网格公式

```css
grid-template-columns: repeat(auto-fill, minmax(20rem, 1fr));
```

## 工作原理

### minmax(20rem, 1fr)

- `min: 20rem` - 列最小宽度 320px，防止内容被压缩
- `max: 1fr` - 剩余空间平分，列可弹性拉伸

### auto-fill

自动计算能容纳多少列：

```
容器 1280px → 4 列 × 320px
容器 960px  → 3 列 × 320px
容器 640px  → 2 列 × 320px
```

## Tailwind 宽度单位

Tailwind 基于 `0.25rem` 刻度：

| 类名 | 计算 | 像素值 |
|------|------|--------|
| `w-80` | 80 × 0.25rem = 20rem | 320px |
| `w-64` | 64 × 0.25rem = 16rem | 256px |
| `gap-4` | 4 × 0.25rem = 1rem | 16px |

## 实际应用

```html
<div class="grid gap-4"
     style="grid-template-columns: repeat(auto-fill, minmax(20rem, 1fr))">
  <div class="w-80">卡片内容</div>
  <!-- 更多卡片... -->
</div>
```

> [!TIP]
> 这种方式比媒体查询更灵活，浏览器自动处理响应式。
