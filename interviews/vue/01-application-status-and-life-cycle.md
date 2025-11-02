# 前端应用状态和生命周期

> 当用户登录成功后，刷新一下页面，为什么就又变回未登录状态了？你该如何解决这个问题？

## 涉及的知识点

这个问题涉及的知识点如下：

* **SPA 的生命周期**： 你是否理解“刷新页面”=“JavaScript 内存被完全清空，所有代码（包括 main.ts）重新执行”？
* **状态管理的本质**： 你是否知道 Pinia/Vuex/Redux 默认都只是内存状态？
* **浏览器存储机制**： 你是否知道 localStorage、sessionStorage 和 cookies 的区别？
  * **localStorage**：持久存储，刷新、关闭浏览器都不丢。
  * **sessionStorage**：会话存储，刷新不丢，但关闭标签页就丢了。
  * **cookies**：会自动附加在 HTTP 请求头中发送给服务器（这既是优点也是缺点）。
* **Token 认证流程**： 你是否知道前端实现“保持登录”的完整流程？
* **HTTP 请求**： 你是否知道 axios 这种库设置的 defaults.headers 也是内存状态，刷新也会丢失？

## 好的回答

默认情况下，Pinia/Vuex 的状态保存在 JavaScript 内存中，刷新页面会导致内存清空，状态丢失。

为了解决这个问题，我们需要将认证凭证（如 JWT Token）持久化。

持久化 Token： 在用户登录成功时，我们不仅要将 Token 存入 Pinia Store，还应该使用 localStorage 将其保存到浏览器中。

恢复状态 (Rehydration)： 在应用每次启动时（即 `main.ts` 运行时），我们需要去 `localStorage` 检查是否存在 `Token`。如果存在，就将其读出并恢复到 `Pinia Store` 中，这样 `isAuthenticated` 就会变回 `true`。

**（加分项）恢复请求头**： 同样在应用启动时，如果从 localStorage 恢复了 Token，我们还必须重新设置 `axios` 的全局 `Authorization Header`。因为这个 Header 配置也是临时的，刷新会丢失，如果不重设，刷新后的 API 请求都会因为缺少 Token 而失败（401 Unauthorized）。

**（加加分项）使用插件**： 为了简化这个流程，我们可以使用 `pinia-plugin-persistedstate` 这样的插件，它会自动帮我们完成第1步和第2步。但第3步（恢复 `axios Header`）通常仍需要我们在 `main.ts` 中手动完成。