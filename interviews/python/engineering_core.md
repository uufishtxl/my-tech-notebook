# Python 工程化与基础

## ⭐️ 如何管理 Python 的项目依赖

* `requirements.in`：手动维护，只包含项目直接依赖的包。
* `requirements.txt`：通过 `pip-compile`自动生成，包含所有直接和间接依赖的精确版本。
* **工作流程**：
    * 在`requirements.in`文件中添加依赖；
    * 运行`pip-compile`
    * 生成包含哈希值的 `requirements.txt`文件
* **优势**：确保开发、测试、生产环境完全一致

### `requirements.in` 和 `requirements.txt` 有什么区别？

- `requirements.in` 声明直接以来，`requirements.txt`锁定完整依赖数。

### 相比`pip freeze`的优势

`pip freeze`最大的问题是它会捕获整个 Python 环境的状态，包括开发工具、测试框架等非生产依赖，这会导致：

* 依赖混淆
* 环境污染风险
* 更新困难

而 `requirements.in` 声明了项目的真实依赖意图，`pip-compile` 基于此生成确定性的构建结果，既保证了可复现性，又保持了依赖声明的清晰度。 

## ⭐️ 如何在项目中安全地管理敏感信息（如 API Key、数据库密码）

应该使用 `.env` 文件，并将其加入 `.gitignore`。然后使用 `decouple`库的 `config()`函数即可根据当前所处环境自动提取出需要的信息。

### `decouple`的`config`函数是怎么工作的？它和 `os.environ.get()`有什么不同？

* `os.environ.get()`：只能读取系统环境变量，无类型转换，需手动处理默认值
* `decouple.config()`：多源查找（环境变量 → .env文件 → 默认值），自动类型转换，更简洁安全


## Python 的 `class` 带 (`__init__`) 和 `TypedDict` 有什么区别？

* `class`是创建有数据和行为（方法）的对象蓝图。
* `TypedDict`只是一个“类型提示”，用来定义一个普通字典（`dict`）的“形状（Shape）”，类似于 TypeScript 的 `interface`，主要用于静态检查 IDE 提示。不会在 Runtime 进行检查。

作用：

* `class`：需要封装数据+行为
* `TypedDict`：处理 JSON API 响应、配置字典等数据结构。

## 在调用 API 时，如何编写“健壮”的代码？

除了基本的异常处理，健壮的 API 调用需要：

1. 分层捕获：从具体到宽泛
2. 重试机制：对临时性错误自动重试
3. 超时控制：避免无限等待
4. 熔断机制：在服务不可用时快速失败

必须使用`try...except...`块。具体的使用方法是，先捕获具体的错误，比如 `openai.APIError`，然后再用一个宽泛的 `except Exception`来兜底。