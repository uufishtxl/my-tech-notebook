# Django ORM vs SQLAlchemy 查询语法对比

Django 和 SQLAlchemy 是 Python 后端最常用的两个 ORM。Django 使用独特的“双下划线”语法，而 SQLAlchemy 更倾向于使用 Python 原生表达式。

## 1. 过滤查询 (Filtering)

| 操作 | Django ORM | SQLAlchemy (FastAPI 常用) | 备注 |
| :--- | :--- | :--- | :--- |
| **相等** | `.filter(age=18)` | `.where(User.age == 18)` | SQLAlchemy 用 `==` |
| **大于 (`>`)** | `.filter(age__gt=18)` | `.where(User.age > 18)` | Django 用 `__gt`, SQLA 直接用 `>` |
| **大于等于 (`>=`)** | `.filter(age__gte=18)` | `.where(User.age >= 18)` | `e` 代表 equal |
| **小于 (`<`)** | `.filter(age__lt=18)` | `.where(User.age < 18)` | |
| **小于等于 (`<=`)** | `.filter(age__lte=18)` | `.where(User.age <= 18)` | |
| **不等于** | `.exclude(age=18)` | `.where(User.age != 18)` | |
| **包含 (LIKE)** | `.filter(name__contains='x')` | `.where(User.name.like('%x%'))` | |
| **在列表中 (IN)** | `.filter(id__in=[1,2])` | `.where(User.id.in_([1,2]))` | |

## 2. 排序 (Ordering)

| 操作 | Django ORM | SQLAlchemy |
| :--- | :--- | :--- |
| **升序 (ASC)** | `.order_by('age')` | `.order_by(User.age)` 或 `.order_by(User.age.asc())` |
| **降序 (DESC)** | `.order_by('-age')` | `.order_by(User.age.desc())` |

## 3. 示例代码对比

### Django ORM 风格
```python
# 找出 chunk_index 大于 10 的记录，按学习时间倒序排列，取第一个
last_record = StudyProgress.objects.filter(
    chunk_index__gt=10
).order_by('-last_studied_at').first()
```

### SQLAlchemy (2.0+) 风格
```python
# FastAPI 中常用的写写法
stmt = select(StudyProgress).where(
    StudyProgress.chunk_index > 10
).order_by(
    StudyProgress.last_studied_at.desc()
).limit(1)

result = await session.execute(stmt)
last_record = result.scalars().first()
```

## 4. 总结
- **Django**: 约定优于配置，用 magic string (`__`, `-`)，写起来快，但 IDE 推导能力弱（字符串里写错了 IDE 不知道）。
- **SQLAlchemy**: 显式优于隐式，用 Python 表达式，IDE 提示完美，类型安全，但写起来稍微啰嗦一点。
