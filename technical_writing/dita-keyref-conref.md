# DITA keyref 与 conref 对比

#dita #technical-writing

## keyref - 文本变量

**定义位置**：`keys.ditamap`

```xml
<keydef keys="product_version">
  <topicmeta>
    <keywords><keyword>1.0</keyword></keywords>
  </topicmeta>
</keydef>
```

**引用方式**：

```xml
<p>当前版本：<keyword keyref="product_version"/></p>
```

## conref - 结构化内容复用

**定义位置**：`warehouse.dita`

```xml
<note id="note_api_required">API key is required.</note>
```

**引用方式**：

```xml
<note conref="warehouse.dita#warehouse/note_api_required"/>
```

## 对比

| 特性 | keyref | conref |
|------|--------|--------|
| 定义位置 | map 文件 | topic 文件 |
| 复用内容 | 纯文本值 | 完整元素结构 |
| 路径依赖 | 无需路径 | 需相对路径 |
| 适用场景 | 变量（版本号、作者名） | 段落、注释、表格 |

## 选择建议

- 简单文本变量 → **keyref**
- 复杂结构片段 → **conref**

---

## DITAVAL 条件过滤


---

## Audience 条件处理

在元素上添加 `audience` 属性进行标记：

```xml
<step audience="developer">...</step>
<p audience="user">...</p>
<note audience="developer user">混合受众</note>
```

### Oxygen 中使用

1. 打开 **Transformation Scenarios**
2. 在 **Filters** 选项卡中选择 `.ditaval` 文件
3. 运行转换

```xml
<!-- user-only.ditaval -->
<val>
  <prop action="exclude" att="audience" val="developer"/>
  <prop action="include" att="audience" val="user"/>
</val>
```

命令行使用：

```bash
dita -i bookmap.ditamap -f html5 --filter=user-only.ditaval
```
