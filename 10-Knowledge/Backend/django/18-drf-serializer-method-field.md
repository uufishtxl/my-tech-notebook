# Django REST Framework: SerializerMethodField

## 核心作用
用于在序列化结果中添加 **Model 上不存在的动态计算字段**。

## 典型场景
1. **状态判断**：根据当前用户判断 `has_liked`。
2. **格式化数据**：将时间戳转为特定格式字符串。
3. **关联计算**：返回关联对象的计数（如 `comments_count`）。

## 代码示例
```python
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    # 1. 定义字段，read_only 默认为 True
    is_adult = serializers.SerializerMethodField()
    
    # 2. 定义 get_<field_name> 方法
    def get_is_adult(self, obj):
        if not obj.age:
            return False
        return obj.age >= 18

    class Meta:
        model = User
        fields = ['id', 'username', 'age', 'is_adult']
```

## 注意事项
- 这个字段默认是 **Read-only** 的。
- 命名必须要匹配：字段名 `xxx` -> 方法名 `get_xxx`。
