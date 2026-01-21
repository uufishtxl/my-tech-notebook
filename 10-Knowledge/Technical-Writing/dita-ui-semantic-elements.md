# DITA UI 与语义元素

#dita #technical-writing #semantic

DITA 是为**机器设计的写作标准语言**，而非视觉设计语言。使用语义元素而非格式元素（如 `<b>`）。

## 菜单路径

```xml
<menucascade>
  <uicontrol>File</uicontrol>
  <uicontrol>Save</uicontrol>
</menucascade>
```

**作用**：
- **机器翻译** - 翻译工具查软件词汇表，确保与界面一致
- **自动化索引** - 自动抓取所有 `<uicontrol>` 生成界面元素索引

---

## UI 元素对照表

| 元素 | 用途 | 示例 |
|------|------|------|
| `<uicontrol>` | 界面控件 | `<uicontrol>Save</uicontrol>` |
| `<menucascade>` | 多级菜单 | File → Save |
| `<wintitle>` | 窗口标题 | `<wintitle>Settings</wintitle>` |
| `<filepath>` | 文件路径 | `<filepath>/usr/local/bin</filepath>` |
| `<codeph>` | 行内代码 | `<codeph>npm install</codeph>` |
| `<codeblock>` | 代码块 | 多行代码 |
| `<varname>` | 变量名 | `<varname>$HOME</varname>` |
| `<userinput>` | 用户输入 | `<userinput>yes</userinput>` |
| `<systemoutput>` | 系统输出 | `<systemoutput>Done.</systemoutput>` |

---

## Note 类型

```xml
<note type="tip">Press Ctrl+S to save quickly.</note>
<note type="warning">This action cannot be undone.</note>
<note type="restriction">Admin access required.</note>
```

### 完整类型对照表

| type | 用途 | 图标 |
|------|------|------|
| `note` (默认) | 一般性说明 | ℹ️ |
| `tip` | 使用技巧 | 💡 |
| `important` | 重要信息 | ⚠️ |
| `warning` | 警告（可能导致问题） | ⚠️ |
| `caution` | 注意（可能数据丢失） | 🔶 |
| `danger` | 危险（人身伤害） | 🔴 |
| `restriction` | 功能限制 | 🚫 |
| `trouble` | 故障排除提示 | 🔧 |
| `remember` | 需要记住的要点 | 📌 |
| `attention` | 需要注意的事项 | ⚡ |
| `fastpath` | 快捷方式 | ⏩ |
| `other` | 自定义 (配合 `othertype`) | - |
