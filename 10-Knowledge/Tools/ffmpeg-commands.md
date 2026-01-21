# `ffmpeg`命令

需要在系统重安装 `ffmpeg`，并将它加入环境变量。最好使用管理员权限。
## 等分切割

```Bash
ffmpeg -i "C:\Users\martin\Downloads\s10_e12.MP3" -f segment -segment_time 60 -c copy "segmented_audio_%03d.mp3"
```

* `-i` - 指定输入文件
* `-f segment` 告诉 `ffmpeg`使用“分段复用器”
* `-segment_time 60` 设置每个分段时长为 60 秒
* `-c copy` 直接“流复制”音频数据，无需重新编码，用于提高速度和保持质量
* `<output>%03d.mp3`：指定输出文件的命名模式。`%03d` 是一个占位符，`ffmpeg`会用一个从0开始，用零填充到三位数的序列号来替换它。
## 指定时间段切割

```Bash
ffmpeg -i "C:\Users\martin\Downloads\s10_e12.MP3" -ss "00:00:00" -to "00:01:59" -c copy -y "s10_e12-cleared_59.MP3"
```
* `-ss "00:00:00"` 开始时间
* `-to "00:01:59"` 结束时间
* `-y` 强制覆盖已存在的输出文件，不进行提示
