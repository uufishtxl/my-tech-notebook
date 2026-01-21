# ffmpeg 音视频切割命令

#ffmpeg #tools #audio

## 按固定时长分割

```bash
ffmpeg -i "input.mp3" -f segment -segment_time 60 -c copy "output_%03d.mp3"
```

| 参数 | 说明 |
|------|------|
| `-f segment` | 启用分割模式 |
| `-segment_time 60` | 每段 60 秒 |
| `-c copy` | 流复制（不重新编码，速度快） |
| `%03d` | 输出文件名序号（001, 002...） |

---

## 按时间段切割

```bash
ffmpeg -i "input.mp3" -ss "00:00:00" -to "00:01:59" -c copy -y "output.mp3"
```

| 参数 | 说明 |
|------|------|
| `-ss` | 开始时间 |
| `-to` | 结束时间 |
| `-c copy` | 流复制 |
| `-y` | 强制覆盖已存在的文件 |

---

## `-c copy` 的作用

- **不重新编码**：直接复制音频流
- **速度极快**
- **无质量损失**

> [!WARNING]
> 流复制可能导致精确度问题（关键帧对齐）。如需精确切割，去掉 `-c copy`。
