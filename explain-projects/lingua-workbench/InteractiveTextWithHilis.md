# InteractiveTextWithHilis 组件

#lingua-workbench #vue #component

交互式高亮文本显示组件，用于渲染带有高亮标记和 AI 听觉乐谱的文本。

## Props

| Prop | 类型 | 说明 |
|------|------|------|
| `text` | string | 原始文本内容 |
| `hilights` | array | 高亮列表，每个元素含 `id`, `start`, `end` |
| `currentActiveId` | string | 当前激活的高亮 ID |
| `analysisResults` | object | AI 分析结果 |

## Emits

| Event | 说明 |
|-------|------|
| `click-highlight` | 点击高亮区域 |

## 核心方法

### `getScriptSegments`

获取 AI Sound Analysis 的听觉乐谱数据（数组）。

### `parseSoundDisplay`

通过正则格式化听觉乐谱显示文本：
- `[]` 包围的片段 `isGhost: true`，表示因发音特征而略去的音节

## Computed

### `tokens`

把纯文本转换成 Token 数组：

```typescript
interface Token {
  text: string       // Token 字符串
  isHighlight: boolean
  data?: HighlightData
  segments?: ScriptSegment[]  // 听觉乐谱片段
}
```
