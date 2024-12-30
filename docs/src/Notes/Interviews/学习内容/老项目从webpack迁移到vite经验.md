{
"[python]": {
"editor.formatOnType": true
},
"window.zoomLevel": 1,
"editor.formatOnSave": true,
"editor.formatOnPaste": true,
"liveServer.settings.donotShowInfoMsg": true,
"C_Cpp.default.compilerPath": "D:\\MinGW\\bin\\gcc.exe",
"editor.unicodeHighlight.allowedCharacters": {
"锟 �": true
},
"explorer.confirmDelete": false,
"git.autofetch": true,
"emmet.includeLanguages": {
"javascript": "javascriptreact",
"vue-html": "html"
},
"[typescript]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[markdown]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[javascript]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[jsonc]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[vue]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"editor.inlineSuggest.suppressSuggestions": true,
"[scss]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"git.confirmSync": false,
"cody.autocomplete.enabled": true,
"[javascriptreact]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"[typescriptreact]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"javascript.updateImportsOnFileMove.enabled": "always",
"gitlens.gitCommands.skipConfirmations": [
"fetch:command",
"stash-push:command",
"switch:command"
],
"[json]": {
"editor.defaultFormatter": "esbenp.prettier-vscode"
},
"files.autoGuessEncoding": true
}

- Vite5 基于原生的 ES 模块，默认支持的浏览器版本是现代浏览器，如 Chrome>=87、Firefox>=78、Safari>=14 等。如果项目需要支持旧版本的浏览器，可能需要使用 polyfill 来提供对旧版本浏览器的兼容性支持。

解决方案：传统浏览器可以通过插件@vitejs/plugin-legacy 来支持，它将自动生成传统版本的 chunk 及与其相对应 ES 语言特性方面的 polyfill。兼容版本的 chunk**只会**在不支持原生 ESM 的浏览器中进行按需加载

- 在 Babel 配置文件中删去使用 webpack 时配置的预设，添加'@vue/cli-plugin-babel/preset'

这个插件都做了哪些事？

主要是以下三点：

为最每个生成的 ESM 模块化方式的 chunk 也对应生成一个 legacy chunk，同时使用 @babel/preset-env 转换（没错，Vite 的内部集成了 Babel），生成一个 SystemJS 模块，关于 SystemJS 可以看点击这里查看，它在浏览器中实现了模块化，用来加载有依赖关系的各个 chunk。
生成 polyfill 包，包含 SystemJS 的运行时，同时包含由要兼容的目标浏览器版本和代码中的高级语法产生的 polyfill。
生成 <script nomodule> 标签，并注入到 HTML 文件中，用来在不兼容 ESM 的老旧浏览器中加载 polyfill 和 legacy chunk。
如此可见，Vite 兼容低版本浏览器的能力就是来自于 @babel/preset-env 无疑了，都是生成 polyfill 和语法转换， 但是这不就和 webpack 一样了么，事实是 Vite 又帮我们多做了一层，那就是上面反复提到的原生浏览器模块化能力 ESM。
