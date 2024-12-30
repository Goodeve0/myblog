# 老项目从 webpack 迁移到 vite 经验

### 什么是 Vite

Vite 是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：

- 一个开发服务器，它基于 **原生 ES 模块** 提供了丰富的内建功能，如速度快到惊人的**模块热更新（HMR）**。
- 一套构建指令，它使用 **Rollup**打包你的代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

### Vite 带来的收益

1. 开发环境启动服务器速度快  
   Vite 在一开始将应用的模块分为依赖和源码两类，改进开发服务器启动时间。

   - 依赖 Vite 使用 esbuild 预构建依赖。esbuild 使用 Go 编写，并且比以 JS 编写的打包器预构建依赖快 10-100 倍。（esbuild 用 go 语言编写，可以充分利用多核 CPU 的优势，而 JS 是单线程运行，比如 webpack 是基于 node.js，所以在这个阶段不如 Vite 快）
   - 源码 包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS）时常会被编辑且不需要所有源码同时被加载。Vite 以原生 ESM 方式提供源码，实际上让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入代码，即只在当前屏幕上实际使用时才会被处理。

### 迁移中问题记录

#### 替换

1. HtmlWebpackPlugin 变量处理

   - 在 vite 中使用插件 vite-plugin-html 来替换

2. 动态导入功能不同

   - 使用 import.meta.glob 替换 require.context

3. CSS 处理方式不同

   - Webpack 使用 css-loader 和 style-loader 处理 CSS 文件
   - Vite 原生支持 CSS，无需任何额外配置。

4. 静态资源处理方式不同

   - Webpack 通过 file-loader 或 url-loader 加载资源
   - Vite 使用内置的静态资源处理功能，无需额外的配置。

#### 兼容性

1. CommonJS 不识别  
   **问题描述：**
   Vite 原生支持 ES Modules，但很多旧项目的依赖库仍使用 CommonJS（如某些未更新的 npm 包）。如果这些库不支持 ESM，直接迁移到 Vite 时会报错，比如 require is not defined。

   - 在 vite 中使用插件 vite-plugin-require-transform 或@originjs/vite-plugin-commonjs 来实现快速转换
     | 特性 | vite-plugin-require-transform | @originjs/vite-plugin-commonjs |
     |-------------------------------|--------------------------------------------------|---------------------------------------------------|
     | **主要用途** | 支持在 Vite 项目中使用 `require` 语法 | 将 CommonJS 模块转换为 ES 模块 |
     | **支持的语法** | 主要针对 `require` 和 `module.exports` | 支持完整的 CommonJS 语法 |
     | **典型场景** | 适合仅需兼容少量 `require` 使用的项目 | 用于迁移大量依赖于 CommonJS 的项目 |
     | **性能开销** | 开销较低，转换范围较小 | 可能较高，需要对 CommonJS 代码进行全面转换 |
     | **适配生态** | 适合 Vite 环境中的小规模兼容需求 | 适用于更复杂的模块系统，兼容性更强 |

   综合项目情况考虑之后选择@originjs/vite-plugin-commonjs 方案。插件再构建过程时需要进行语法转换，可能会增加一定的性能开销，小项目可以考虑手动替换。

2. 浏览器兼容性问题

   - Vite5 基于原生的 ES 模块，默认支持的浏览器版本是现代浏览器，如 Chrome>=87、Firefox>=78、Safari>=14 等。如果项目需要支持旧版本的浏览器，可能需要使用 polyfill 来提供对旧版本浏览器的兼容性支持。

   解决方案：传统浏览器可以通过插件@vitejs/plugin-legacy 来支持，它将自动生成传统版本的 chunk 及与其相对应 ES 语言特性方面的 polyfill。兼容版本的 chunk**只会**在不支持原生 ESM 的浏览器中进行按需加载

   - 在 Babel 配置文件中删去使用 webpack 时配置的预设，添加'@vue/cli-plugin-babel/preset'

这个插件都做了哪些事？

主要是以下三点：

为最每个生成的 ESM 模块化方式的 chunk 也对应生成一个 legacy chunk，同时使用 @babel/preset-env 转换（没错，Vite 的内部集成了 Babel），生成一个 SystemJS 模块，关于 SystemJS 可以看点击这里查看，它在浏览器中实现了模块化，用来加载有依赖关系的各个 chunk。
生成 polyfill 包，包含 SystemJS 的运行时，同时包含由要兼容的目标浏览器版本和代码中的高级语法产生的 polyfill。
生成 `<script nomodule>` 标签，并注入到 HTML 文件中，用来在不兼容 ESM 的老旧浏览器中加载 polyfill 和 legacy chunk。
如此可见，Vite 兼容低版本浏览器的能力就是来自于 @babel/preset-env 无疑了，都是生成 polyfill 和语法转换， 但是这不就和 webpack 一样了么，事实是 Vite 又帮我们多做了一层，那就是上面反复提到的原生浏览器模块化能力 ESM。
