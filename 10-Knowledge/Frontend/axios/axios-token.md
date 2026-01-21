# Axios Token 管理

> 👨：你用 Vue 和 Django 做登录，请描述一下你的前端会如何处理这个流程。

> 👩：我会用 Pinia 存 Token。在 login 动作成功的那一刻，我立刻会设置 axios.defaults.headers.common['Authorization']。这样，axios 实例就被“注入”了认证信息，之后所有的 API 调用都会自动带上这个请求头，这遵循了 DRY (Don't Repeat Yourself) 原则，非常优雅且不易出错。在 logout 时，我再 delete 掉这个 default 即可。