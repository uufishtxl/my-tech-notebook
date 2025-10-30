# HTTP 安全规范

> 👨：你用什么方式来做 API 认证？

> 👩：我用 JWT Token。

> 👨：那这个 Token 是怎么在 HTTP 请求中传递的？

> 👩：我把它放在 Authorization 请求头 (Request Header) 中，使用 Bearer 规范 (Bearer Scheme)。格式是：Authorization: Bearer <token_string>。