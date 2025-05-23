# 老项目从 Webpack5 迁移到 Vite 经验

## Vite

### 什么是 Vite

Vite 是一种新型前端构建工具，能够显著提升前端开发体验。它主要由两部分组成：

- 一个开发服务器，基于 **原生 ES 模块** 提供了丰富的内建功能，如 **模块热更新（HMR）**。
- 一套构建指令，使用 **Rollup** 打包代码，并且它是预配置的，可输出用于生产环境的高度优化过的静态资源。

### Vite 带来的收益

1. **开发环境启动服务器速度快**

   - **依赖：** 使用 esbuild 预构建依赖，比传统打包器预构建依赖快 10-100 倍，因为 esbuild 用 Go 语言编写，支持多核并行，而 Webpack 是基于单线程的 Node.js，所以在这个阶段不如 Vite 快。

   ![alt text](image.png)

   - **源码：** 包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS）时常会被编辑且不需要所有源码同时被加载。Vite 采用原生 ESM 方式提供源码，实际上让浏览器接管了打包程序的部分工作：仅在浏览器请求时处理。动态加载当前所需内容，从而减少不必要的资源加载和处理。

   ![alt text](image-1.png)

   ![alt text](image-2.png)

2. **几乎实时的模块热更新**

   Webpack 热更新需要重新编译所有模块，然后启动服务器，换句话来说 Webpack 的热更新速度和初次编译启动时间相差不多。这会给开发者带来一些负面的体验，比如修改组件样式后可能需要等待很长时间页面才会重新渲染。

3. **所需文件按需编译，避免编译用不到的文件**

## 迁移中问题记录

### 插件修改

#### 动态注入

- 在 Webpack 配置中，使用 `HtmlWebpackPlugin` 来动态注入 HTML 模板变量：

  ```javascript
  new HtmlWebpackPlugin({
    template: "./src/index.html", // 指定模板文件
    filename: "index.html", // 生成文件名
  });
  ```

- 在 Vite 中使用插件 `vite-plugin-html` 来替换：

  ```javascript
  createHtmlPlugin({
    inject: {
      data: {
        title: "Vite Example",
      },
    },
  });
  ```

#### CSS 处理方式不同

- Webpack 使用 `css-loader` 和 `style-loader` 处理 CSS 文件：

  ```javascript
  rules: [
    {
      test: /\.css$/,
      use: ["style-loader", "css-loader"],
    },
  ];
  ```

- Vite 原生支持 CSS，无需任何额外配置。

#### 静态资源处理

- Webpack 通过 `file-loader` 或 `url-loader` 加载资源：

  ```javascript
  test: /\.(png|jpg|gif|svg)$/,
  use: [
    {
      loader: 'file-loader',
      options: {
        name: 'assets/[name].[hash].[ext]',
      },
    },
  ];
  ```

- Vite 使用内置的静态资源处理功能，无需额外的配置。

### 配置变更

#### 别名配置

根据 Vite 别名规则，`vite.config.js` 添加配置，把 `@/` 指向 `src` 目录：

```javascript
resolve: {
  alias: [
    {
      find: '@/ ',
      replacement: '/src/',
    },
  ],
},
```

#### 添加后缀

由于引入组件没有携带文件后缀 `.vue` 会报错，有两种解决方案：

1. 手动添加 `.vue` 后缀，但项目庞大，很多地方都没有带后缀，全部改动难度较大。
2. 配置 `vite.config.js` 的 `extensions` 字段，来自动查找文件扩展名后缀：

   ```javascript
   {
     extensions: [".vue", ".js", ".json"];
   }
   ```

#### 动态导入

Webpack 中 `require.context` 示例代码：

```javascript
const modules = require.context("./modules", false, /\.js$/);
modules.keys().forEach((key) => {
  const module = modules(key);
  console.log(module.default); // 模块的默认导出内容
});
```

Vite 使用 `import.meta.glob` 替换：

```javascript
const modules = import.meta.glob("./modules/*.js");
for (const path in modules) {
  modules[path]().then((module) => {
    console.log(module.default); // 模块的默认导出内容
  });
}
```

`import.meta.glob` 只有在 `modules['./modules/example.js']()` 被调用时，浏览器才会加载对应文件，避免一次性加载所有模块减少启动时的资源消耗。比起 require.context，提供了更灵活的加载方式，可以按需加载模块，提升性能。

#### CSS 全局变量

项目在 `less` 文件中定义了变量，并在 Webpack 配置中通过 `style-resources-loader` 设置为全局变量：

```javascript
{
  loader: 'style-resources-loader',
  options: {
    patterns: ['src/styles/var.less'],
  },
},
```

在 `vite.config.js` 中添加如下配置：

```javascript
css: {
  preprocessorOptions: {
    less: {
      additionalData: `@import "src/styles/var.less";`
    },
  },
},
```

#### 环境变量

Vite 对环境变量的访问需要通过 `import.meta.环境变量名称`，将 `process.env` 替换为 `import.meta.env`。

环境变量在 `.env` 文件里配置，变量名必须以 `VITE_` 开头。

### 兼容性处理

#### CommonJS 不识别

##### 问题描述

Vite 原生支持 ES Modules，但很多旧项目的依赖库仍使用 CommonJS。如果这些库不支持 ESM，迁移到 Vite 时会报错，比如 `require is not defined`。

##### 解决

在 Vite 中使用插件 `vite-plugin-require-transform` 或 `@originjs/vite-plugin-commonjs`。以下是两者的对比：

| 特性           | vite-plugin-require-transform          | @originjs/vite-plugin-commonjs             |
| -------------- | -------------------------------------- | ------------------------------------------ |
| **主要用途**   | 支持在 Vite 项目中使用 `require` 语法  | 将 CommonJS 模块转换为 ES 模块             |
| **支持的语法** | 主要针对 `require` 和 `module.exports` | 支持完整的 CommonJS 语法                   |
| **典型场景**   | 适合仅需兼容少量 `require` 使用的项目  | 用于迁移大量依赖于 CommonJS 的项目         |
| **性能开销**   | 开销较低，转换范围较小                 | 可能较高，需要对 CommonJS 代码进行全面转换 |
| **适配生态**   | 适合 Vite 环境中的小规模兼容需求       | 适用于更复杂的模块系统，兼容性更强         |

根据项目需求，选择 `@originjs/vite-plugin-commonjs`。插件再构建过程时需要进行语法转换，可能会增加一定的性能开销，小项目可以考虑手动替换。

#### 浏览器兼容性

##### 问题描述

Vite5 基于原生的 ES 模块，默认支持的浏览器版本是现代浏览器（如 Chrome >= 87、Firefox >= 78、Safari >= 14）。

##### 解决

通过插件 `@vitejs/plugin-legacy` 支持传统浏览器，它将自动生成传统版本的 chunk 及与其相对应 ES 语言特性方面的 polyfill。兼容版本的 chunk**只会**在不支持原生 ESM 的浏览器中进行按需加载。

示例配置：

```javascript
legacy({
  targets: ["ie >= 9"],
  additionalLegacyPolyfills: ["regenerator-runtime/runtime"],
});
```

#### 依赖兼容问题

##### 问题描述

项目依赖中有部分缺乏维护的个人项目，这些项目中会存在部分不符合 ESM 语法，Vite 运行时会报错，因此需要对依赖库进行适配。

##### 解决

1. 使用 monorepo，将依赖库代码 fork 到项目中成为一个子工程，对子工程的代码进行适配，然后作为项目源码链接到主项目中。
2. 将直接库代码复制到项目中，但可能由于依赖变动导致异常。
3. 替换为由大团队维护的知名库。

#### CSS 前缀问题

##### 问题描述

原项目配置了 pocss-loader 来打包时自动加 CSS 前缀以兼容低版本浏览器，迁移到 Vite 后也需要处理该问题。

##### 解法

在 Vite 中可以采用`autoprefixer`来实现。安装`autoprefixer`依赖后，在 `vite.config.js` 的 `css.postcss.plugins` 里面添加 autoprefixer 插件。由于该插件默认只支持`CommonJS`，所以要用 `require` 引入，后面配置要支持的目标浏览器：

```javascript
css: {
  postcss: {
    plugins: [
      require("autoprefixer")({
        overrideBrowserslist: [
          "Android 4.1",
          "iOS 7.1",
          "Chrome > 31",
          "ff > 31",
          "ie >= 9",
          "> 1%",
        ],
        grid: true,
      }),
    ];
  }
}
```

### 优化相关

#### 代码分割

##### 问题描述

代码分割不起效

##### 解决办法

首先确保在`.env`文件中设置了`VITE_USE_SPLIT_CHUNKS`为`true`，其次检查是否正确使用`import()`语法进行代码分割。解决完代码分割后发现部分代码块过大，进而使用 chunk-size 控制代码块的大小，当然也可以动态导入来按需加载代码块。

### 部分原理剖析

#### vite 的 HMR 和 webpack 的 HMR

##### **Vite 的 HMR**

- **基于 ES 模块：** Vite 使用原生的浏览器支持的 ES 模块 (`ESM`) 来实现 HMR。当文件发生更改时，Vite 会通过开发服务器发送 WebSocket 消息，通知浏览器仅重新加载受影响的模块及其依赖模块。
- **直接提供源码：** 文件改动后，Vite 的开发服务器将只发送修改后的模块内容，浏览器根据模块依赖关系自动重新导入模块，无需额外的打包。

##### **Webpack 的 HMR**

- **基于打包：** Webpack 的 HMR 依赖其打包过程。在开发模式下，Webpack 将生成一个包含模块更新逻辑的 HMR runtime，同时会注入一些特殊的 API（如 `module.hot`），用于接收和处理模块的更新。
- **模块热更新：** 当模块发生变化时，Webpack 的开发服务器会生成一个增量更新文件（`update.json`），将其通过 WebSocket 通知客户端，然后客户端根据 runtime 替换更新的模块。

#### @vitejs/plugin-legacy

@vitejs/plugin-legacy 是一个 Vite 插件，这个插件内部同样使用 `@babel/preset-env` 以及 `core-js`等一系列基础库来进行语法降级和 Polyfill 注入，以解决在旧版浏览器上的兼容性问题，它主要做了以下几件事：

1. **配置解析**：插件会读取用户的配置，确定需要支持的浏览器版本和需要应用的 Polyfill。
2. **Babel 转译**：在构建过程中，插件会使用`@babel/preset-env`将现代 JavaScript 代码转译为兼容旧版浏览器的代码。它会根据配置的目标浏览器版本自动选择需要的 Babel 插件和预设。
3. **Polyfill 注入**：插件会根据代码中使用的特性和目标浏览器的支持情况，自动注入必要的 Polyfill。通常使用 `core-js`或`regenerator-runtime`来实现这一功能。
4. **Chunk 分离**：为了优化加载性能，插件会将现代代码和转译后的代码分离成不同的 chunk。现代浏览器会加载未转译的代码，而旧版浏览器会加载转译后的代码。
5. **动态加载**：通过在 HTML 中插入条件性脚本标签，插件确保浏览器根据其能力选择加载合适的代码版本。
6. **环境变量注入**：在生产环境中 `import.meta.env.LEGACY` 变量为 `true`。
