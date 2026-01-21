# DITA Bookmap 结构元素

#dita #technical-writing #bookmap

## Preface 前言

```xml
<preface href="topics/frontmatter/preface.dita"/>
```

- `preface` 中的 `section` 在全书目录层级中重要性较低
- 前言被视为独立的介绍性单元

> [!NOTE]
> `notices` 同样使用专门标签，`section` 也不进入主目录。

---

## booklists 的默认隐藏

**现象**：`<booklists>` 内的 `<toc>`、`<figurelist>`、`<tablelist>` 不会出现在目录中（目录里不会有"目录"条目）。

**需求**：如需让这些列表标题出现在 PDF 书签中，需自定义。

---

## abstract 和 shortdesc

这些摘要性内容被视为主题**元数据**，默认不生成独立目录项。

---

## notices 版权声明

包含版权、商标等法律文本，位置和样式有固定要求，一般不进入主目录。

---

## 控制 PDF TOC 的方法

### 推荐：基于 outputclass 属性

```xml
<bookmap>
  <frontmatter>
    <booklists outputclass="toc"/>
    <preface outputclass="toc">
      <topicref href="preface.dita"/>
    </preface>
  </frontmatter>
</bookmap>
```

### 实现方式

| 方式 | 适用场景 |
|------|----------|
| 自定义 XSLT | DITA-OT 旧版 PDF 引擎 |
| 自定义 CSS | PDF2 基于 CSS 的引擎 |
| Oxygen 配置 | 商用工具图形界面 |

---

## Part 组织

```xml
<part navtitle="Getting Started">
  <chapter href="ch1.dita"/>
  <chapter href="ch2.dita"/>
</part>
```

**DITA-OT 自动处理**：
1. **自动插入分页** - 生成显眼的隔页
2. **自动编号** - 显示 "Part I. Getting Started"

> [!TIP]
> 用 `navtitle` 是因为不需要正文内容。需要正文时可用 `href` 引入 concept 文件。
