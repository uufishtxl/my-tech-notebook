# DITA Map vs Bookmap

#dita #technical-writing #map

## 对比

| 特性 | `<map>` | `<bookmap>` |
|------|---------|-------------|
| 结构 | 扁平/简单树状 | 强制书籍结构 |
| 元素 | 只有通用 `<topicref>` | 专门插槽：frontmatter, chapter, appendix |
| 适用 | 简单文档 | **PDF 手册**（企业最常用） |

## map - 简单结构

```xml
<map>
  <topicref href="topic1.dita"/>
  <topicref href="topic2.dita">
    <topicref href="topic2a.dita"/>
  </topicref>
</map>
```

## bookmap - 书籍结构

```xml
<bookmap>
  <frontmatter>
    <preface href="preface.dita"/>
  </frontmatter>
  <chapter href="ch1.dita"/>
  <chapter href="ch2.dita"/>
  <appendix href="appendix.dita"/>
  <backmatter>
    <indexlist/>
  </backmatter>
</bookmap>
```

> [!TIP]
> bookmap 类似前端的"企业级路由"，有严格的分区（Dashboard、Setting、User）。

---

## Profiling 条件过滤

DITA 支持对内容打标签，通过配置文件过滤：

| 属性 | 示例值 |
|------|--------|
| `platform` | windows, linux, macos |
| `product` | pro, lite, enterprise |
| `audience` | admin, user, developer |

---

## Appendix 内容类型选择

| 内容 | 推荐 Topic Type |
|------|-----------------|
| 快捷键列表 | Reference (`r_keyboard_shortcuts.dita`) |
| 技术栈表格 | Reference (`r_tech_stack.dita`) |
| 术语表 | Glossentry (`ge_sound_script.dita`) |
| 版本历史 | Topic (通用) |
