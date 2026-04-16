
## `QuerySet` 是惰性的

所谓的 **“惰性 (Lazy)”**，用大白话解释就是：**不见兔子不撒鹰**，或者说 **极其严重的拖延症**。QuerySet 是惰性的，只有在迭代、切片、`list()`、`len()` 等操作时才真正执行 SQL。

![[Pasted image 20260201212300.png]]

上面的 `QuerySet` 比较简单，但是即便是这样又是过滤、又是排序的语句，也是同样的道理：
```Python
books = Book.objects.filter(title__contains="Book").order_by('-id')
```

