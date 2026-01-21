---
index: 11
gemini_id: 378fb0da904fe47b
synced_at: 2026-01-20T11:49:26.606Z
---

## Q

sqlalchemy 我又忘记了。我记得要 declarative base，然后要有个engine，然后要用ORM，然后有commit flush 等等，你等帮我大概梳理下，帮助我回忆么

## A

没问题，SQLAlchemy 的概念确实比较多，很容易弄混。不用担心，我们用一个\*\*“从下往上”**的逻辑，配合一个**“购物”的类比\*\*来帮你快速找回记忆。

你可以把整个流程想象成在**餐馆点菜**：

### 1\. 核心组件 (The Big Picture)

按顺序来，这三个是必须的“基础设施”：

1.  **Engine (引擎/后厨):**
    
    *   这是最底层的连接。它负责和数据库（MySQL, PostgreSQL, SQLite）真正“说话”。
        
    *   它维护连接池（Connection Pool）。
        
    *   _代码:_ `engine = create_engine('sqlite:///example.db')`
        
2.  **Declarative Base (基类/菜单模板):**
    
    *   这是 ORM 的核心。你需要一个基类，然后所有的“表”（Table）都继承这个类。
        
    *   它把 Python 的类（Class）映射成数据库的表（Table）。
        
    *   _代码:_ `class Base(DeclarativeBase): pass` (SQLAlchemy 2.0 写法) 或 `Base = declarative_base()` (旧写法)。
        
3.  **Session (会话/服务员):**
    
    *   **这是你平时操作最多的对象。**
        
    *   它是一个“暂存区”或“手写单”。你对数据的增删改查，都在 Session 里发生，直到你确认提交。
        
    *   它实现了“工作单元（Unit of Work）”模式。
        

* * *

### 2\. 操作流程 (The Workflow)

我们用一个标准的流程把它们串起来：

#### 第一步：建基建 (Setup)

你需要先定义好 Engine 和 Base，如果有表还没建，要通过 metadata 创建。

```
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import DeclarativeBase, sessionmaker

# 1. 引擎
engine = create_engine('sqlite:///:memory:') # 这里用内存数据库演示

# 2. 基类
class Base(DeclarativeBase):
    pass

# 3. 定义模型 (ORM)
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String)

# 4. 在数据库中建表 (emit CREATE TABLE)
Base.metadata.create_all(engine)

# 5. 造一个 Session 工厂 (Session类)
SessionLocal = sessionmaker(bind=engine)

```

#### 第二步：日常操作 (Daily Usage)

每次你要操作数据库，你都要**实例化一个 Session**。

```
# 开启一个会话 (叫来了服务员)
session = SessionLocal()

# --- 新增 ---
new_user = User(name="Gemini")
session.add(new_user)  # 此时还没写入数据库，只是写在了服务员的小本本上

# --- 查询 ---
# 2.0 风格查询
stmt = select(User).where(User.name == "Gemini")
user = session.scalars(stmt).first()

```

* * *

### 3\. 重点难点：Commit vs Flush

这是你特意提到的，也是最容易混淆的地方。

*   **Session (暂存区)**: 你 `session.add(obj)` 或者修改了对象的属性 `obj.name = "New"`，这时候数据**还在 Python 内存里**，数据库完全不知道。
    
*   **Flush (冲刷/同步)**:
    
    *   **动作**: 把 Python 内存里的变动，翻译成 SQL 语句（INSERT/UPDATE），发送给数据库执行。
        
    *   **状态**: 数据库执行了 SQL，**事务（Transaction）还没提交**。
        
    *   **结果**: 此时如果你去数据库查（在同一个事务内），能看到数据；**主键 ID 此时生成了**。但如果程序崩溃，这些数据会丢失。
        
    *   _通常你不需要手动调 flush()，查询或 commit 前它会自动触发。_
        
*   **Commit (提交/落袋为安)**:
    
    *   **动作**: 发送 `COMMIT` 指令给数据库。
        
    *   **状态**: 数据库把刚才 Flush 过去的一堆 SQL 操作永久保存。
        
    *   **结果**: 其他连接/用户现在能看到这些数据了。
        
*   **Rollback (回滚)**:
    
    *   撤销本次 Session 中所有未 Commit 的操作（包括已经 Flush 的）。
        

**一句话总结区别：**

> **Flush** 是把改动告诉数据库（还在事务中，由于拿到了数据库生成的ID，此时对象有了 `id`）； **Commit** 是让数据库把这些改动永久生效（事务结束）。

* * *

### 4\. 简易记忆口诀

为了帮你回忆，记住这个顺序：

1.  **Engine** 是路（连数据库）。
    
2.  **Base** 是图（定义表结构）。
    
3.  **Session** 是手（用来干活）。
    
4.  先 **Add** 到手，再 **Flush** 换 ID，最后 **Commit** 也就是存。
    
5.  用完记得 **Close**（或者用 `with` 上下文管理器）。
    

这一遍梳理清楚了吗？如果你的代码是基于 `AsyncIO` (异步) 的，流程大体一样，只是关键字变成了 `await session.commit()`。需要我写个简单的完整 demo 给你 copy 备用吗？
