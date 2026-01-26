---
name: daily-quiz
description: 每日技术测验与间隔复习系统，将错题转为次日简答题
---

# Daily Quiz Skill

## 功能说明
为用户生成前端/后端/DITA 技术测验，并将错题加入次日复习队列。

## 工作流程

### 1. 生成每日测验
当用户请求测验时：
- 每个分类（前端/后端/DITA）生成 5 道题
- 题型混合：选择题、是非题、简答题
- **重要**：先检查 `pending_review.md` 是否有待复习的错题，如有，将其作为简答题优先出题

### 2. 批改与记录
批改后：
- 将错题和"蒙对的"知识点记录到 Obsidian vault 的 `review/` 文件夹
- 将错题摘要追加到 `.gemini/skills/daily-quiz/pending_review.md`

### 3. 间隔复习
下次测验时：
- 读取 `pending_review.md` 中的待复习项
- 将其转化为简答题出题
- 用户答对后，从待复习列表移除

## 文件结构
```
.gemini/skills/daily-quiz/
├── SKILL.md              ← 本文件
└── pending_review.md     ← 待复习错题列表
```

## 待复习列表格式 (pending_review.md)
```markdown
## 待复习错题

### 后端
- [ ] 解释 ACID 事务属性的含义
- [ ] GIL 对 I/O 密集型和 CPU 密集型任务的影响有何不同？
- [ ] 什么是幂等性？哪些 HTTP 方法是幂等的？

### 前端
- [ ] position: sticky 会脱离文档流吗？与 absolute/fixed 有何区别？
- [ ] HTTP 304 状态码表示什么？

### DITA
(暂无)
```

## 使用方式
用户可以说：
- "给我来一份每日测验"
- "帮我复习昨天的错题"
- "把这道错题加入复习列表"
