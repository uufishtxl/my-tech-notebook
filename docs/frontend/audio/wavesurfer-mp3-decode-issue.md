# wavesurfer.js 无法加载某些"有效"的 MP3

#frontend #audio #wavesurfer #troubleshooting

## 问题

一个看起来能被 VLC 或其他播放器正常播放的 MP3 文件，在 `wavesurfer.js` 中加载时却抛出 `TypeError`。

## 原因

`wavesurfer.js` 依赖浏览器内置的 `AudioContext.decodeAudioData` API 来解码音频。这个 API 的规范非常严格，对于一些含有以下问题的 MP3 文件，可能会解析失败：

- 非标准元数据
- 复杂 VBR（可变比特率）头信息
- 嵌入了大量无关数据（如巨大的专辑封面）

相比之下，`ffmpeg` 或 VLC 等专业工具的解码器容错性要强得多。

## 解决方案："净化" MP3 文件

使用 `ffmpeg` 的流复制命令来重新封装文件：

```bash
ffmpeg -i <input.mp3> -c copy <output.mp3>
```

这个命令在复制音频流的同时，会：
- 丢弃掉不规范的元数据和头部
- 重新生成一个结构干净、高度标准化的新文件

这个"净化"过后的文件可以被浏览器解码器毫无问题地解析。

---

> [!NOTE]
> `-c copy` 参数表示"流复制"，不会重新编码音频，因此速度非常快且不会有质量损失。

*记录于 2025-11-07*
