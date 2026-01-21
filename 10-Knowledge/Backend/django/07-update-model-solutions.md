# `Model` 为已有数据的 Model 添加“非空无默认值”字段

## 问题的根源

为已有数据的 `Model` 添加 `null=False` 字段时，必须解决存量数据的填充问题。‘删除数据库’是开发初期放弃数据的一种极端做法；而在实际工作中，标准的‘改造工程’是通过 `makemigrations` 提供一次性默认值，或在 `models.py` 中指定 `default` 属性来安全地完成数据迁移。

**核心原因**： 数据库在执行 `ALTER TABLE ... ADD COLUMN ...` 时，不知道应该为那些已经存在的行填充什么值到这个新列。由于新列被约束为 `NOT NULL`，而我们又没提供默认值，数据库无法完成操作。

当需要添加新的不能为空（`null=False`）且没有默认值字段至既有 `Model` 时，最干脆利落的方法是将当前db直接删除。

## 不同的改造过程

### 方案一：【开发阶段】推到重来

* **操作**： 直接删除数据库文件（例如 `db.sqlite3`）或清空整个 database。
* **描述**： 这是在项目极早期开发阶段、数据完全不重要时的“权宜之计”。它通过彻底删除所有数据，绕过了“存量数据如何填充”的问题。
* **严重警告**： 这在生产环境 (Production) 或任何有价值数据的环境中是绝对禁止的，因为它会导致毁灭性的数据丢失。

1. 在 PowerShell 上用 `Remove-Item`指令清除当前db。
2. 清除掉App当前的 `migrations` 蓝图。
3. 修改 Models。
4. 重新 `makemigrations` 和 `migrate`。
5. 重新建立 superuser 账号。
6. 重新设置 Sites 中的 `Domain Name` 字段。

### 方案二：利用 `makemigrations` 的智能提示

### 方案三：在 Model 中指定 `default` 属性
