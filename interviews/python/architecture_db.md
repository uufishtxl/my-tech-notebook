# 软件架构与数据库

## ⭐️⭐️ 解释下 MVC、MVP 和 MVVM 模式的区别

都是为了解耦。MVC 是经典模式，但 V 和 M 有耦合。MVP 用 Presenter 彻底解耦了 V 和 M，但 P 很“臃肿”。MVVM (如 Vue) 用“数据绑定”技术自动化了 MVP 的手动更新，效率最高。

### 更细化的理解

* MVC
  * 用户操作交给 Controller，由它更新 Model，再通知 View 刷新。
  * View 需要知道 Model 的结构来渲染界面，这导致了二者的耦合
* MVP
  * 用 Presenter 作为中间人，彻底切断了 View 和 Model 的直接联系。
  * View 变得“愚蠢”，只负责显示 UI 和转发用户输入
  * Presenter 持有 View 和 Model 的引用，承担了所有协调的工作。
  * 遗留问题：手动同步，Presenter 需要编写大量代码来更新 View，导致它变得臃肿。
* MVVM
  * 引入 ViewModel 和数据绑定
  * ViewModel 是 Model 的抽象，暴露了 View 需要的状态和命令。
  * 数据绑定引擎自动同步 View 和 View Model 的状态，无需手动更新。

## ⭐️ 对比下 Django 和 FastAPI

Django 是“全家桶” (Opinionated)，自带 ORM, Admin, Forms，适合快速开发内容管理系统。FastAPI 是“乐高积木” (Unopinionated)，只负责 API 和数据验证 (Pydantic)，性能极高，为异步而生，需要自己组合 ORM（如 SQLAlchemy）。

## ⭐️ 为什么要用 `utf8mb4`，而不是 `utf8`

因为 MySQL 的 `utf8` 是个“假”的，只支持3字节，存不了 Emoji (表情符号)。`utf8mb4` 才是真正的 UTF-8，支持4字节。