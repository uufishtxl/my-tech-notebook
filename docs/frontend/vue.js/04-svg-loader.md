# Vite + Vue 3 图标管理方案：`unplugin-icons`

这篇笔记的目标是配置一套“一次配置、永久享受”的图标工作流。相比 `svg-sprite-loader`，其配置更迅速，不需要自己编写 SVG 图标组件。

最终效果：

1. **无需 `import`**：在模板中直接使用 `<i-mdi-home />即可渲染图标。
2. **按需加载**：只有用到的图标才会被打包，极致优化性能。
3. **本地支持**：可以将自己的 `src/icons`目录下的 SVG 当做图标组件使用（如 `<i-local-my-logo />
4. **IDE友好**：VSCode（或其他 IDE）会自动补全图标名，且 TypeScript 不会报错。
5. **样式可控**：可以用 `font-size` 和 `color`像控制文字一样控制图标的大小和颜色。

## 步骤一：安装依赖

```Bash
# -D 表示安装为开发依赖 (devDependencies)
npm install unplugin-icons unplugin-vue-components -D
```

* `unplugin-icons`：核心组件，负责识别、查找和编译图标
* `unplugin-vue-components`：辅助插件，负责自动导入 Vue 组件。是它让我们免去了手写 `import` 的步骤。

<table>
  <td>
    💡 你不需要手动安装任何图表集（如 `@iconify/json-mdi`）。`unplugin-icons`默认启用了 `autoInstall: true`，它会在你首次使用某个图表及（如 `mdi` 时自动帮忙安装）.
  </td>
</table>

## 步骤二：配置 `vite.config.js`

这是最关键的一步。我们将配置两个插件协同工作。

```JavaScript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// 1. 导入所需插件
// 负责自动导入组件 (包括图标)
import Components from 'unplugin-vue-components/vite'
// 负责图标本身
import Icons from 'unplugin-icons/vite'
// 负责解析图标 (IconsResolver)
import IconsResolver from 'unplugin-icons/resolver'
// 负责加载你本地的 SVG 文件
import { FileSystemIconLoader } from 'unplugin-icons/loaders'

// 你本地存放 SVG 图标的文件夹路径
const localIconPath = './src/icons'

export default defineConfig({
  plugins: [
    vue(),

    // --- 自动导入组件配置 (unplugin-vue-components) ---
    Components({
      // 核心：自动生成 TypeScript 类型定义文件
      // 它会在根目录生成 components.d.ts
      // 这就是 VSCode 能识别图标组件的关键
      dts: true,

      // 核心：配置解析器
      resolvers: [
        // --- 图标解析器 (unplugin-icons/resolver) ---
        IconsResolver({
          // 约定前缀：所有以 'i-' 开头的组件都会被识别为图标
          // 例如 <i-mdi-home /> 会被解析
          // 如果你改为 'icon', 那么就要写 <icon-mdi-home />
          prefix: 'i',

          // 启用我们自定义的本地 SVG 图标集
          // 'local' 是我们给这个图标集起的名字
          // 对应下面 Icons 插件中的 customCollections
          customCollections: ['local'],

          // (可选) 自动安装图标集，默认为 true
          // enabledCollections: ['mdi', 'ant-design'] // 如果你只想开启特定的图标集
        }),
      ],
    }),

    // --- 图标插件配置 (unplugin-icons) ---
    Icons({
      // 编译器：告诉 unplugin-icons 如何编译图标
      // 'vue3' 表示编译为 Vue 3 兼容的组件
      // (还有 'react', 'svelte' 等)
      compiler: 'vue3',

      // (可选) 自动安装图标集
      // 当检测到代码中使用了 'mdi' 等图标集时，会自动 npm install @iconify/json-mdi
      autoInstall: true,

      // 核心：配置自定义本地图标集
      customCollections: {
        // 'local' 是图标集的名字，必须和上面 IconsResolver 中的一致
        local: FileSystemIconLoader(
          // 本地 SVG 图标的存放路径
          localIconPath,
          
          // (可选) 对 SVG 进行转换
          // 这个函数会在加载每个 SVG 时执行
          // 这里我们给每个 SVG 强制添加了 fill="currentColor"
          // 这使得我们可以通过 CSS 的 'color' 属性来控制图标颜色
          svg => svg.replace(/^<svg /, '<svg fill="currentColor" ')
        ),
      },
    }),
  ],
})
```

## 步骤三：配置 TypeScript（tsconfig.app.json）

**目的**：让 TypeScript 识别自动生成 `components.d.ts`文件，消除 IDE 报错并提供类型提示。

**重要提示**：不要修改根目录的 `tsconfig.json`！Vite 项目使用 `references`来组织 TS 配置，App 相关配置在 `tsconfig.app.json`中。

打开`tsconfig.app.json`（位于项目根目录）：

```JSON
// tsconfig.app.json
{
  "extends": "@vue/tsconfig/tsconfig.dom.json",

  // 核心：在这里加入 'components.d.ts'
  "include": [
    "env.d.ts",
    "src/**/*",
    "src/**/*.vue",
    "components.d.ts" // <-- 确保这一行被添加
  ],
  
  "exclude": ["src/**/__tests__/*"],
  "compilerOptions": {
    "composite": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

操作完成后，重启 VSCode。

## 步骤四：使用图标

现在，你可以在任何 .vue 文件的 <template> 中直接使用图标，无需 import。

### 使用开源图标 (Iconify)

可以去 [Icones](https://icones.js.org/) 网站查找图标。

**命名规则**： `<[前缀]-[图标集]-[图标名] />`

* **前缀**: `i` (我们在 `vite.config.js` 中配置的)
* **图标集**: `mdi` (Material Design Icons), `fa-solid` (Font Awesome Solid), `ant-design` 等
* **图标名**: `home`, `user-circle`, `setting-filled` 等

```HTML
<template>
  <i-mdi-home />

  <i-fa-solid-user />

  <i-ant-design-setting-filled />
</template>
```

### 使用本地 SVG 图标

在 `src/icons` (在 `vite.config.js` 中配置的路径) 放入你的 SVG 文件，例如 `my-logo.svg`。

**命名规则**： `<[前缀]-[自定义图标集名]-[文件名] />`

* **前缀**: `i`
* **图标集名**: `local` (在 `vite.config.js` 中配置的)
* **文件名**: `my-logo`

```HTML
<template>
  <i-local-my-logo />
</template>
```

## 控制图标样式（大小和颜色）

### 控制大小

推荐使用 `font-size`

```HTML
<template>
  <i-mdi-home style="font-size: 48px;" />

  <i-mdi-home class="text-3xl" />
</template>
```

也可以使用 `width/height`，但是 fonts-size 更符合排版流。

### 控制颜色

直接使用 CSS 的 `color` 属性。

**原理**：

* **开源图标**： 默认带有 `fill="currentColor"`。
* **本地图标**： 我们在 `vite.config.js` 中通过 `svg.replace` 强制添加了 `fill="currentColor"`。

`currentColor` 会自动抓取 CSS 的 `color` 属性。

```HTML
<template>
  <i-mdi-home style="color: red;" />
  <i-local-my-logo style="color: #42b883;" /> <i-mdi-thumb-up class="text-blue-500" />
</template>
```

> [!TIP]
> ✨ 如果本地 SVG 文件内部（例如 <path> 标签写死了 `fill="#000"`，这个硬编码颜色的优先级更高，会导致 `color` 样式失效。需要确保本地 SVG 是干净的，移除了内部颜色定义。

## 总结：打包（`npm run build`）

当你执行 `npm run build` 时，Vite 和 `unplugin-icons` 会自动扫描你的代码，只把 `<i-mdi-home />` 这样实际用到的图标编译成内联 SVG 并打包进最终的 JS 文件中。

这实现了完美的按需加载，性能远超于打包整个字体文件或大型雪碧图的旧方案。

## 与雪碧图旧方案的本质区别

**`svg-sprite-loader` 的工作流是“雪碧图 (Sprite Sheet)”**

* 它把你导入的所有 SVG 文件，打包合并成一个巨大的 `symbol-defs.svg` 文件（一个包含很多 `<symbol>` 标签的雪碧图）。
* 这个雪碧图文件通常会被自动注入到你 `index.html` 的 `<body>` 标签顶部。
* 你写的 `SvgIcon.vue` 组件通过 `<use xlink:href="#icon-name">` 语法，来**引用**雪碧图中已经定义好的 `<symbol>`。
* **缺点**：你必须在入口文件一次性导入所有图标，导致这个雪碧图文件很大，无论当前页面是否用到了所有图标，用户都必须下载它。

**`unplugin-icons` 的工作流是“按需内联 (On-demand Inline SVG)”**

* 它根本不会生成雪碧图。
* 它在编译时扫描你的 `.vue` 模板，当它看到 `<i-mdi-home />` 时，它会想：“哦，用户需要 `mdi` 的 `home` 图标”。
* 然后它会动态地、即时地生成这个图标对应的 Vue 组件代码（代码内容大致就是 `<svg>...</svg>`）。
* 这个 `<i-mdi-home />` 标签最终在浏览器 DOM 中渲染出来的是完整的 `<svg>...</svg>` 标签，而不是 `<use>` 标签。
* 优点 (核心)：这实现了终极的按需加载 (Tree-Shaking)。只有你实际使用的图标，才会被编译和打包进你当前页面的 JS chunk 中。你没用过的图标，体积为零。