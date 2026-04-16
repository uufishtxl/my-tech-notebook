## 进度

- [2026-02-26]：`reader/views.py` #67

## Todo
- 对 `batch_translate` 使用 `with transaction.atomic()`，通过事务隔绝来实现“0 或全”。