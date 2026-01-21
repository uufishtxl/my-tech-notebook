# DITA Reference 与数据管理

#dita #technical-writing #reference

Reference = **数据库**，只存事实（参数、规格、错误码），不包含步骤。

## Reference 核心结构

```xml
<reference id="ref_example">
  <title>...</title>
  <refbody>
    <refsyn>...</refsyn>      <!-- 命令语法 -->
    <properties>...</properties>  <!-- 键值对 -->
    <table>...</table>        <!-- 复杂表格 -->
  </refbody>
</reference>
```

---

## CALS Table（复杂表格）

支持合并单元格、列宽定死、表头跨页重复：

```xml
<table>
  <tgroup cols="2">
    <colspec colname="c1" colwidth="1*"/>
    <colspec colname="c2" colwidth="1*"/>
    <thead>
      <row><entry>Parameter</entry><entry>Value</entry></row>
    </thead>
    <tbody>
      <row><entry>Voltage</entry><entry>220V</entry></row>
    </tbody>
  </tgroup>
</table>
```

### colspec 属性

| 属性 | 说明 |
|------|------|
| `colwidth="1*"` | 类似 CSS `1fr` |
| `colwidth="8cm"` | 固定宽度 |
| `colsep="0/1"` | 右边框线 |
| `rowsep="0/1"` | 下边框线 |

> [!TIP]
> 别手写 CALS Table，用 Python 脚本从 JSON/Excel 自动生成。

---

## properties（轻量键值对）

比 Table 简单，适合环境配置、错误码列表：

```xml
<properties>
  <property>
    <proptype>Voltage</proptype>
    <propvalue>220V</propvalue>
  </property>
</properties>
```

---

## refsyn（命令语法）

用于描述命令行语法、API 签名：

```xml
<refsyn>
  <cmdname>django-admin</cmdname>
  <kwd>startproject</kwd>
  <var>project_name</var>
</refsyn>
```

> [!WARNING]
> 除非必要，优先用 `<codeblock>`，`refsyn` 过于复杂。

---

## Warehouse 策略

建立不发布的 `warehouse.dita` 存放可复用内容：

```xml
<!-- warehouse.dita -->
<ph id="product_name">Lingua Workbench</ph>
<note id="note_api_required">API key is required.</note>
```

其他文件通过 conref 引用：

```xml
<ph conref="warehouse.dita#warehouse/product_name"/>
```

**价值**：Single Source of Truth（一处修改，全局更新）
