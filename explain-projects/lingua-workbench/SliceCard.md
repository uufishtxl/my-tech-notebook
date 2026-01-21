# SliceCard 组件

#lingua-workbench #vue #component

音频片段卡片组件，用于显示和编辑单个音频 Slice 的信息。

## Props

| Prop | 类型 | 说明 |
|------|------|------|
| `url` | string | Chunk 音频 URL |
| `start` | number | 片段起始时间 |
| `end` | number | 片段结束时间 |
| `region` | object | 包含 `id`, `start`, `end`, `originalText` |
| `initialHighlights` | array | 后端返回的高亮笔记数据 |
| `initialPronunciationHard` | boolean | 是否标记为发音难 |
| `initialIdioms` | boolean | 是否标记为习语 |

## Emits

| Event | 说明 |
|-------|------|
| `delete` | 删除片段 |
| `adjust-start` | 向前调整 start |
| `adjust-end` | 向后调整 end |
| `update-markers` | 更新标记状态 |

## 核心状态 (ref)

| Ref | 说明 |
|-----|------|
| `currentSlice` | 包含 `text` 和 `highlights` 列表 |
| `activeHighlightId` | 当前 Focus 的高亮文本 ID |
| `isEditingOriginal` | 是否正在编辑文本 |
| `selectedTextInfo` | 选中文本信息（text, start, end, rect） |
| `isPlaying` | 是否正在播放 |
| `isLooping` | 是否循环播放 |

## 核心方法

| 方法 | 触发时机 | 功能 |
|------|----------|------|
| `handleTextSelection` | 释放鼠标 | 获取选区信息，显示高亮图标 |
| `createHighlight` | 点击高亮图标 | 创建新高亮区域 |
| `handleDeleteHighlight` | HighlightEditor emit | 删除高亮 |
| `handleAiResult` | HighlightEditor emit | 更新 AI 分析结果 |
| `handleSaveData` | HighlightEditor emit | 保存分析数据 |
| `getSliceData` | expose 给父组件 | 构建 server-ready 数据 |

## Watch

| 监听目标 | 行为 |
|----------|------|
| `props.region.originalText` | 同步到 `currentSlice.text` |
| `currentSlice.value.text` | 清理高亮和分析数据 |
| `activeHighlightId` | 适时停止录音 |
| `currentPlaybackRate` | 调整播放速率 |

## Computed

| 名称 | 说明 |
|------|------|
| `dynamicTextStyle` | 根据文本长度动态计算字体大小 |
| `activeHighlight` | 当前 Focus 的高亮对象 |
| `savedAnalysisForActive` | 当前高亮的 Sound Analysis |
| `savedDictionaryForActive` | 当前高亮的 Dictionary Analysis |
