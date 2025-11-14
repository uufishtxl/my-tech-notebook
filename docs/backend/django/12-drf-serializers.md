# Django > 序列化器

序列化器会：

* 序列化：将Python对象（如模型实例、字典）转换为一种可传输的格式（如 JSON 字符串）
* 反序列化：将传入的、可传输格式的数据（如 JSON 字符串）转换回 Python 的内部数据类型，并且在转换过程中，进行验证，检查：
	* 字段是否齐全
	* 数据类型是否正确
	* 是否符合其他规则（`max_length`等）
	* 甚至可以执行自定义验证

## 序列化

从 Python 数据转换为 JSON 字符串的过程，到底发生了什么？

* 准备一个 Python 对象（比如模型实例或者字典，或者它们组成的列表）
* 创建 `Serializer`实例：`Serializer`实例是一个 Python 对象。
* 访问 `.data`属性（序列化过程）
	```Python
	serialized_python_data = serializer.data
	```
	- 当访问 `serializer.data`时，序列化器会执行序列化操作。
	- 如果准备的 Python 对象是一个列表，会遍历每一个模型实例
	- 根据序列化器的定义（`Meta.fields`），将每个模型实例转换为一个 Python 字典。
	- 最终，serialized_python_data 会是一个 Python 字典或 Python 字典的列表。
	也就是说，serialized_python_data 此时仍然是 Python 原生数据结构，而不是 JSON 字符串。
* 构建 `Response`对象：`Response`对象接收 `serialized_python_data`作为其主要参数。
* 渲染为 JSON 字符串：在 `Response`对象将 HTTP 响应发送到网络之前，它会使用一个“渲染器”，默认情况下是 `JSONRenderer`，它会将 Response 对象中包含的 Python 数据结构转换为一个 JSON 格式的字符串。这个 JSON 字符串就是最终发送给前端的 HTTP 响应体。

> [!Tip] 当序列化数据为一个列表时，需要传入 `many=True`，告诉序列化器准备处理一个对象列表。

可以看到，正是因为 Response 最好接收的是一个Python的数据类型，而模型实例内部复杂，需要序列化后，才会转换为一个满足这种要求的数据结构并存入到序列化器实例中的 `data`属性中。所以才会产生上述的动作。

如果准备好的数据是通过 ORM 查询到的 `QuerySet`，则一个好的习惯是先将它通过 `list()`转换为列表，再传入 `Response`对象中。

如果数据本身就是一个Python 的字典或列表，则直接传入 `Response`对象即可。
