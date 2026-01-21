# Mermaid CLI 使用指南

#tools #mermaid #diagram

官方命令行工具，将 Mermaid 图表转换为 PNG/SVG/PDF。

## 安装

```bash
npm install -g @mermaid-js/mermaid-cli
```

## 基本用法

```bash
# Mermaid 文件 → PNG
mmdc -i input.mmd -o output.png

# 带白色背景
mmdc -i input.mmd -o output.png -b white

# 转 SVG（矢量图）
mmdc -i input.mmd -o output.svg

# 转 PDF
mmdc -i input.mmd -o output.pdf
```

## 常用参数

| 参数 | 说明 |
|------|------|
| `-i` | 输入文件 |
| `-o` | 输出文件 |
| `-b` | 背景色 (`white`, `transparent`, `#hex`) |
| `-t` | 主题 (`default`, `forest`, `dark`, `neutral`) |
| `-w` | 输出宽度 (px) |
| `-H` | 输出高度 (px) |

## 为什么用 CLI？

- ✅ **免费** - mermaid.live 导出需付费
- ✅ **批量处理** - 可脚本化
- ✅ **本地执行** - 不依赖网络
- ✅ **CI/CD 集成** - 构建流程中自动生成

## 技术原理

底层使用 Puppeteer（无头 Chrome）渲染，质量与网页版一致。
