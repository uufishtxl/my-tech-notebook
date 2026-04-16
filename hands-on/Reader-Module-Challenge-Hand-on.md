# Reader 模块手写挑战计划

## 1. 核心目标
将 `reader` 模块中的核心逻辑（文章导入、生词管理、AI 辅助阅读）通过“手写练习”的方式在 `sandbox` 模块中复现，加深对复杂业务逻辑、异步任务和 AI 继承的理解。

---

## 2. 挑战内容预估

### A. 文章管理 (**Articles**)
- **Model**: `Article` (标题、原文、解析后的 HTML、大纲、来源等)。
- **View**: 文章列表分页、详情展示。
- **Action**: 文章标记已读、文章删除逻辑。

### B. AI 解析流程 (The "Secret Sauce")
- **Schema**: 定义 LLM 返回的 JSON 结构 (Pydantic)。
- **Task**: 异步解构文章（提取关键词、生成大纲、翻译）。
- **Service**: 封装 OpenAI/Anthropic API 调用逻辑。

### C. 词汇追踪 (Vocabulary)
- **Model**: `UserWord` (单词、熟悉度、掌握状态)。
- **Link**: 文章内容与用户词库的实时关联渲染。

---

## 3. 下一步行动
1. 分析 `reader/models.py` 确定核心字段。
2. 分析 `reader/views.py` 确定核心 API 行为。
3. 在 `sandbox/models.py` 和 `sandbox/views.py` 中初始化挑战代码。

*最后更新: 2026-02-28*


## 4. CRUD 视图重构检查清单 (`/api/v1/articles/`)
- [ ] `GET` - list (仅当前用户数据)
- [ ] `GET` - retrieve (深度嵌套阅读：文章 -> 段落 -> 词语)
- [ ] `POST` - create (触发 Huey 异步提取、总结任务)
- [ ] `PUT` - update

*(注：涉及 Django Model、DRF Serializers 和 ViewSets 的深度原理总结已归档拆分至 `10-Knowledge/Backend/django/` 目录下)*