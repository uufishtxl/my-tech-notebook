# Vite 前端代理

#vite #frontend #proxy #cors

## 功能

在 `vite.config.js` 中配置代理，将前端请求转发到后端：

```javascript
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true
      }
    }
  }
}
```

**效果**：
- 前端 `http://localhost:5173/api/users`
- 转发到 `http://localhost:8000/api/users`

## 工作原理

```
浏览器 → Vite (5173) → 后端 (8000)
         ↑ 同源         ↑ 代理转发
```

浏览器认为请求的是 5173（同源），不触发跨域检查。

## ⚠️ 局限性

| 环境 | 解决方案 |
|------|----------|
| **开发环境** | Vite Proxy ✅ |
| **生产环境** | 后端 CORS 配置 ✅ |

> [!IMPORTANT]
> Vite 代理**只在开发环境有效**！
> 生产环境需要后端配置 CORS：
> ```python
> # Django settings.py
> CORS_ALLOWED_ORIGINS = ["https://your-frontend.com"]
> ```
