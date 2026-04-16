> **问题**：在 DRF Serializer 中，`validate_<field_name>` 和 `validate` 方法的执行顺序是什么？

> 为什么要知道这个知识点：是为了**省钱（性能）和防爆（代码安全）**。

**场景**：你要验证一个邀请码。

1. **格式检查**（免费）：必须是 6 位数字。
    
2. **数据库检查**（昂贵）：必须是数据库里存在的有效码。

```Python
class PromoCodeSerializer(serializers.Serializer):
    code = serializers.CharField()

    # 1. 【前哨战】validate_code (廉价操作)
    # 目的：如果是垃圾格式，直接踢回去，别去骚扰数据库。
    def validate_code(self, value):
        if len(value) != 6 or not value.isdigit():
            raise serializers.ValidationError("格式错误") # 🚫 在这就拦住了！
        return value

    # 2. 【大决战】validate (昂贵操作)
    # 只有当上面格式正确时，才会执行这里。
    def validate(self, attrs):
        code = attrs['code']
        # 💸 下面这行要查库，还要算钱，很贵！
        # 如果不知道顺序，甚至可能因为 data['code'] 不存在而报错
        if not PromoCode.objects.filter(key=code).exists():
            raise serializers.ValidationError("不存在的邀请码")
        return attrs
```

- **目的 1：Fail Fast (快速失败/省资源)**
    - **单字段校验**通常只是 CPU 算一下（正则、长度），很快。
    - **全局校验**通常涉及跨表查询、数据库 IO，很慢。
    - **结论**：利用顺序，**先用廉价规则过滤掉 90% 的垃圾请求**，保护你的数据库。
- **目的 2：信任数据 (避免 KeyError)**
    - 一旦进入 `validate(self, attrs)`，你可以 100% 放心：**只要 attrs 里有的字段，肯定都是格式正确的**。
    - **结论**：你不需要在全局校验里再次检查 `if age < 0`，因为 `validate_age` 早就帮你把好关了。
#### `serializers.Serializer` 和 `serializers.ModelSerializer` 的区别

* **A. 笨办法 (`serializers.Serializer`)** 你必须**手动**做所有事：定义每个字段、自己写存进数据库的代码。
* **聪明办法 (`serializers.ModelSerializer`)** DRF 帮你**自动**把 Model 映射成 Serializer，并送你 `create/update` 方法。

写 `validate` 方法的时候，数据类型就是普通的 Python 字典。