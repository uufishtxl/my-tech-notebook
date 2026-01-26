# DITA 复习笔记

## 2026-01-22 测试记录

### ✅ 全部答对！保持状态 🎉


## 2026-01-23 测试记录

### ✅ 已掌握
- xref 交叉引用
- shortdesc 位置（在 title 之后）
- topicref href 指向 Topic 文件

### ❌ 待加强
- **warning vs caution**：
  - Warning = 可能导致**人身伤害**（更严重）
  - Caution = 可能导致**设备/数据损坏**（较轻）
- **keyref vs conref**：
  - keyref = 引用 Key 定义（变量，在 Map 中定义）
  - conref = 直接引用内容块

### 知识点巩固

#### Topic 类型选择
| 类型 | 适用场景 | 关键词 |
|------|---------|--------|
| **Concept** | 解释"是什么" | 概念、背景、原理 |
| **Task** | 说明"怎么做" | 步骤、操作、安装 |
| **Reference** | 查阅"参数表" | API、配置项、语法 |

#### conref 内容复用
- 允许在多个 Topic 中复用同一段内容
- 修改源内容，所有引用处自动更新

#### DITA Map 作用
- 组织和排列多个 Topic 的结构关系
- 定义导航、层级、输出顺序

#### 条件处理 (Conditional Processing)
- **步骤 1**：在内容中添加属性（如 `audience="developer"` 或 `platform="windows"`）
- **步骤 2**：创建 `.ditaval` 文件定义 include/exclude 规则
- **步骤 3**：发布时应用 ditaval，按条件过滤内容
