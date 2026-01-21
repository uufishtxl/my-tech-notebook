# Vue v-if vs v-show 与 DOM 文本索引陷阱

#vue #v-if #v-show #dom

## v-if vs v-show

| 指令 | 行为 | DOM 影响 | 适用场景 |
|------|------|----------|----------|
| `v-if` | 条件性渲染 | 销毁/重建元素 | 切换不频繁 |
| `v-show` | 条件性显示 | `display: none` | 频繁切换 |

## v-if 的隐患：注释节点

```html
<!-- v-if 为 false 时，Vue 会留下占位注释 -->
<span>
  <!--v-if-->
</span>
```

**问题**：如果注释周围有换行/空格，浏览器可能将其解释为额外的文本节点，导致：

- `textContent` 长度与预期不符
- `Range.toString()` 结果偏移
- 划词高亮等需要精确索引的功能出 bug

## 解决方案

1. **使用 v-show**：元素始终存在，不产生注释节点
2. **使用 white-space: pre-wrap**：保留原始空白格式

```css
.text-container {
  white-space: pre-wrap;
}
```

> [!WARNING]
> 涉及文本选区和字符索引映射时，优先用 `v-show` 避免 DOM 结构意外变化。

## Selection 和 Range API

```javascript
const selection = window.getSelection()
const range = selection.getRangeAt(0)

range.startContainer  // 起始节点
range.startOffset     // 起始偏移
range.toString()      // 选中文本
```
