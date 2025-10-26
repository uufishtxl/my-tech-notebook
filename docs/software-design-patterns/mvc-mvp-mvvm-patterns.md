# 架构模式对比：MVC vs. MVP vs. MVVM

这三种都是用于分离用户界面(UI)、业务逻辑和数据模型的软件架构模式。它们的共同目标是**解耦 (Decoupling)**，使代码更易于维护、测试和复用。

---

## ⭐️ 面试速答版 (Cheatsheet)

**面试官问：“能谈谈你对 MVC, MVP, MVVM 的理解吗？”**

你可以这样回答：

“它们都是为了解决**界面、逻辑和数据解耦**的架构模式。”

1.  **MVC (Model-View-Controller)** 是最经典的一个。
    * `Model` (数据)、`View` (视图)、`Controller` (控制器)。
    * 它的工作流是：**Controller** 作为入口，接收用户请求，然后去操作 **Model** (数据)，并选择一个 **View** (模板) 来渲染，最后返回给用户。
    * 在经典 MVC 中，View 是**可以**直接访问 Model 数据的。比如 Django (MVT) 里，Template (V) 就可以直接渲染 Model (M) 传来的数据。

2.  **MVP (Model-View-Presenter)** 是 MVC 的一个演进。
    * `Model` (数据)、`View` (视图)、`Presenter` (主持人)。
    * 它最大的区别是：**View 和 Model 完全解耦**，老死不相往来。
    * 所有的通信都必须通过 **Presenter**。Presenter 从 Model 获取数据，然后**主动调用** View 提供的接口（如 `view.showData(data)`) 去更新界面。View 变得非常“被动”和“愚蠢”(Dumb View)，只负责响应 Presenter 的命令。

3.  **MVVM (Model-View-ViewModel)** 是目前前端和移动端的主流。
    * `Model` (数据)、`View` (视图)、`ViewModel` (视图模型)。
    * 它和 MVP 很像，ViewModel 负责逻辑。但它最大的法宝是**数据绑定 (Data-Binding)**。
    * **ViewModel** 只负责暴露数据（如 `userName`）。**View** 会自动“绑定”到这些数据上。当 ViewModel 里的 `userName` 改变时，View 上的文本框**自动更新**，反之亦然。
    * 这极大地减少了 MVP 中那种繁琐的 `view.showData()` 手动调用，让开发者只关心数据和逻辑，不用操心 DOM 操作。

**总结：** 演进的核心就是为了**解耦**和**提升可测试性**。MVP 让 V 和 M 彻底分开；MVVM 用“数据绑定”技术自动化了 MVP 的手动更新，效率最高。

---

## 核心区别对比表

| 模式 | 核心组件 | View/Model 耦合度 | 通信方式 | 适用场景 |
| :--- | :--- | :--- | :--- | :--- |
| **MVC** | Model, View, **Controller** | **耦合** (View 直接访问 Model) | Controller 更新 Model, Controller 选择 View | Web 后端 (Django, Rails) |
| **MVP** | Model, View, **Presenter** | **解耦** (两者互不知晓) | Presenter <-> View (通过接口), Presenter <-> Model | Android, Windows Forms |
| **MVVM**| Model, View, **ViewModel**| **解耦** (两者互不知晓) | View <-> ViewModel (**数据绑定**), ViewModel <-> Model | 前端 (Vue, React), 移动端 |

---

## 深入解析与演进关系

### 1. MVC (Model-View-Controller)

这是 Web 开发的基石。

* **Model (模型):** 数据层。负责数据结构、业务逻辑以及和数据库交互。(e.g., Django 的 `models.py`)
* **View (视图):** 展示层。负责渲染 UI 界面。(e.g., Django 的 `templates.html`)
* **Controller (控制器):** 逻辑层。接收用户输入，解析请求，调用 Model 处理数据，然后把数据交给 View 去渲染。(e.g., Django 的 `views.py` 扮演了这个角色)

**经典问题：Django 自称 MVT？**
Django 说的 MVT (Model-View-Template) 只是换了个说法：
* **Model** = Model
* **Template** = View (负责展示)
* **View** = Controller (负责逻辑)

### 2. MVP (Model-View-Presenter)

MVP 的出现是为了解决 MVC 中 View 和 Model 的耦合问题，这在需要复杂 UI 交互（如桌面应用）时非常致命，且难以测试。

* **Model (模型):** 同上，只管数据。
* **View (视图):S** **被动的 (Passive)** 界面。它只包含最简单的 UI 逻辑（如按钮的颜色），并将所有事件 (如 `onClick`) **转发**给 Presenter。它会暴露一套接口 (Interface) 供 Presenter 调用，如 `void showUserName(String name);`。
* **Presenter (主持人):** 核心逻辑。它作为 View 和 Model 之间唯一的桥梁。它从 Model 获取数据，处理后，调用 View 的接口 `view.showUserName(...)` 来更新 UI。

**优点：** View 和 Model 彻底解耦，**可测试性极高**。你可以轻易地 Mock 一个 View 来测试 Presenter 的所有逻辑。

**缺点：** 随着界面变复杂，Presenter 需要调用的 View 接口会越来越多，代码变得臃肿（**Boilerplate**）。

### 3. MVVM (Model-View-ViewModel)

MVVM 的出现是为了解决 MVP 的“臃肿”问题，它被广泛用于现代 UI 框架 (Vue, React, Angular, WPF, Xamarin)。

* **Model (模型):** 同上，只管数据。
* **View (视图):** UI 层 (e.g., HTML, XAML)。它**声明式地 (declaratively)** 绑定到 ViewModel 上的属性。它**不知道** ViewModel 的具体实现。
* **ViewModel (视图模型):** 核心逻辑。它非常像 Presenter，负责从 Model 获取和处理数据。但它**不持有 View 的引用**。它只是**暴露 (Expose)** 一些属性（如 `userName`）和命令（如 `saveCommand`）。

**核心魔法：数据绑定 (Data-Binding)**
* **View -> ViewModel:** 当用户在文本框里输入时，View 会自动更新 ViewModel 里的 `userName` 属性。
* **ViewModel -> View:** 当 ViewModel 里的 `userName` 属性（比如从网络获取后）发生变化时，View 上的文本框会**自动刷新**。

**优点：** 开发者彻底从繁琐的 DOM 操作 (如 `document.getElementById`) 或 MVP 的 `view.showData()` 中解放出来，**开发效率极高**。