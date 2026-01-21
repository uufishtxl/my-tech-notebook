# frontend > Axios 的一些功能特性

`axios.create({config})`: 创建 axios 实例

## 配置

`withCreadentials` 用来表示是否在跨站请求中使用证书，默认值为`false`

我们这个项目的思路是：

`dj-rest-auth`使用 JWT，并通过`my-auth-cookie`和 `my-refresh-cookie` 存储 Refresh
 Token，刷新端点为 `api/token/refresh`

```Python
# settings.py
REST_AUTH = {
    'USE_JWT': True,
    'JWT_AUTH_COOKIE': 'my-auth-cookie', # (可选) 我们可以把 Token 存在 Cookie 里
    'JWT_AUTH_REFRESH_COOKIE': 'my-refresh-cookie', # (可选)
}
```

```python
# urls.py
urlpatters = [
	# ...
	path('api/token/refresh/', get_refresh_view().as_view(), name='token_refresh'),
	# ...
]
```

这就决定了必须在 axios 实例中配置 `withCredentials` 为 `true` 才可以使用 Cookie 中的令牌。

然后我们可以使用拦截器，在捕获到 401 错误（访问权限问题）后，自动发出刷新令牌请求。

## 我的问题和解答

### `config` 中的 `timeout`，如何针对上传请求进行超时设置
  
解决方案是可以在单次 API 请求中覆盖默认设置。比如在我们 `LoadSource.vue`文件中的 `handleUploadHttpRequest`方法里，调用 `api.post`时多传递一个配置对象即可。

```JavaScript
try {
	await api.post('/v1/audios', formData, {
		headers: {
			'Content-Type': 'multipart/form-data'
		},
		timeout: 120000
	})
	handleUploadSuccess()
} catch (error) {
	ElMessage.error('Upload failed!')
	console.error(error)
}
```
   

### axios 自动转换 JSON 为 JavaScript 数据

实现转换的“幕后功臣”是 `axios`的一个核心内置功能。它的工作原理是：

1. `axios`收到后端 HTTP 响应后，会检查响应头中 `Content-Type`字段。
2. 如果发现 `Content-Type`是 `application/json`，`axios`会自动在内部使用 `JSON.parse()`来处理响应体的原始文本。
3. 这个被解析后的 JavaScript 对象或数组，最终被放到了访问的 `response.data`属性上。

这个行为由 `axios`一个叫做 `transformReponse`的默认配置项控制。如果要处理非 JSON 格式的响应（比如 XML），也可以自定义这个配置来处理。

反之亦然，当使用 `axios.post`, `axios.put`, `axios.patch` 等方法，并且传入的 `data` 参数是一个普通的 JavaScript 对象或数组时， `axios`会自动完成以下操作：

* 自动设置 `Content-Type`：会将请求头中的 `Content-Type`自动设置为 `application/json;charset=utf-8`
* 自动 `JSON.stringify()`，将 JavaScript 对象或数组转换为 JSON 字符串
* 发送 JSON 字符串：将这个 JSON 字符串作为请求体发送出去

例外情况（不会自动 `stringify`）：

* `FormData` 对象：当传入的是一个 `FormData` 实例（比如批处理文件上传时），`axios`不会对其进行 `JSON.stringify()`。它会设置 `Content-Type`为 `multipart/form-data`，并直接发送 `FormData` 对象。
* `URLSearchParams`对象
* 原始类型：如果传入的是一个字符串、数字或布尔值等原始类型，`axios`会直接发送它们
