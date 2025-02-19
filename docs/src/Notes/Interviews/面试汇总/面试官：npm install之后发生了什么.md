# 面试官：npm install 之后发生了什么

小白同学终于迎来了期待已久的二面，然而滴滴面试官的这一问题却让他有些“破防”了。自诩略懂前端工程化的他，听到面试官问“`npm install`  之后究竟发生了什么？”时，陷入了沉默。\
小白同学：“…什么？”\
面试官微微一笑，并给出了**最后的轻语**😭：“那今天的面试就先到这里吧，好吧？”
回到家中，小白同学决定好好研究这个问题，想弄明白“`npm install`  后，究竟发生了什么？\
他开始深入思考，阅读文档，最终发现了其中的奥秘——

## 什么是 npm？

npm（node package manager），是随同 Node.js 一起安装的第三方包管理器。通过 npm，我们可以安装、共享、分发代码，管理项目的依赖关系。

### 嵌套结构

在 npm 的早期版本中，npm 以递归的方式去处理依赖，每个依赖包都会在自己的 `node_modules` 目录中安装其子依赖，直到没有子依赖为止。

举个栗子，我们项目安装 axios 依赖，axios 自身需要三个依赖：

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/b100a347582e44b2ac1e3cf29f3b5892~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=vZbkBUdbtCGhLwPCuJFCgIgpAHA%3D" alt="" width="100%">

而其中的 form-data 又存在三个依赖，这里就不一一列举了，当执行 npm install 命令后，得到的 node_modules 中的模块目录结构：

    my-project
    └── node_modules
        ├── axios
        │   ├── follow-redirects
        │   ├── form-data
        │   │   ├── asynckit
        │   │   ├── combined-stream
        │   │   │   └── delayed-stream
        │   │   └── mime-types
        │   │       └── mime-db
        │   └── proxy-from-env

这样的优点就是 node_modules 的结果和 package.json 的结果一一对应，层级结构明显，而且保证了每次安装目录结构都是相同的。

但也存在不容忽视的缺点：如果项目一旦变大，依赖变多，node_modules 将变得非常庞大，特别是不同层级的依赖如果共用一个依赖，还会造成不必要的冗余，而且整个的嵌套层级也会非常深。

### 扁平结构

为了解决以上的问题，npm3.x 开始将嵌套结构改为了扁平结构，当安装模块时，不管是直接依赖还是子依赖，优先将其安装在 node_modules 根目录

![](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/4cc864b3d78948a3a96fb20b2abe1485~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=7jJyEEjdYXPcZeiZ2EOzn20wsT4%3D)

这样就是一个扁平结构，当安装到相同的时候，判断已安装的是否符合新的版本范围，如果符合就跳过，不符合就继续安装

## 配置文件

**package.json**

一般来说，任何使用 Node.js 的项目都需要有一个 package.json 文件，该文件中包括项目名称、版本、描述和所依赖的包

**package-lock.json**

为了解决 `npm install` 的不确定性问题，在 `npm 5.x` 版本新增了 `package-lock.json` 文件，而安装方式还沿用了 `npm 3.x` 的扁平化的方式。它描述了生成的确切树，以便后续安装能够生成相同的树，而不管中间依赖更新如何。

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/854f99644c2b4be7a7911486e66a673f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=jVq8E0%2BFRmsfYHTnJnNcBrd9dUs%3D" alt="" width="100%">

**.npmrc**

控制 npm 的行为，如注册表、代理、缓存路径等。

    # 包下载源
    registry=https://registry.npmmirror.com

    # 设置作用域包的私有仓库（如公司内部包）
    @mycompany:registry=https://npm.mycompany.com

    # 设置缓存目录路径（默认 ~/.npm）
    cache=~/.custom-npm-cache

    # 设置 HTTP 代理（根据实际情况替换）
    proxy=http://127.0.0.1:8080

`.npmrc` 文件的优先级为：项目级 > 用户级 > 全局级 > 内置级。

## npm 缓存

在执行 `npm install` 或 `npm update` 命令下载依赖后，除了将依赖包安装在 `node_modules` 目录下外，还会在本地的缓存目录缓存一份。我们可以通过以下命令获取缓存位置：

    npm config get cache

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/056c689690814fda9c489b342315b078~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=bak6D2Jg8qI83yPJecdWVw1BKpo%3D" alt="" width="100%">

打开目录有以下文件夹

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a7846cab9a3947db8a4c4e31665257a9~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=cuxBdALM3%2BWD61HRg%2FIS2DGj27Q%3D" alt="" width="100%">

content-v2 存放的是依赖实际的内容，而 index-v5 则是存放依赖的索引信息

## 依赖完整性

在下载依赖包之前，我们一般就能拿到 `npm` 对该依赖包计算的 `hash` 值，例如我们执行 `npm info` 命令，紧跟 `tarball`(下载链接) 的就是 `shasum`(`hash`)

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/55c4498e31e140e29bafeb3ef6c2a086~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=xjIxRoXTERXdNoDEp%2FfluXkWl6E%3D" alt="" width="100%">

在下载依赖包之前，npm 会获取其 shasum 哈希值。下载完成后，npm 会在本地重新计算哈希值，并与远程的哈希值对比。如果两者一致，则依赖包完整；否则，npm 会重新下载。

#### 下载包

如果检查到本地缓存中不存在对应的依赖包,便会通过发送网络请求去下载包。具体是通过 package-lock.json 文件中的 resolved 字段,当我们尝试通过该字段中的值从浏览器中输入,发现会直接给我们下载了一个文件,例如,我们使用 `axios` 中的 `resolved` 中的值,具体值是:

    https://registry.npmjs.org/axios/-/axios-1.3.1.tgz

## 整体流程

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/2b7cc92f27e242ec86f4b798612b0107~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP54ix5ZCM5a2mXw==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTk2MzcxNTc1NDEyMjUyIn0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1740586243&x-orig-sign=CM%2F2kEWWEskOZywqon%2BNP6Lhk78%3D" alt="16f0eef327ccaba5~tplv-t2oaga2asx-jj-mark_3024_0_0_0_q75.webp" width="100%">

1.  首先，`npm install` 需要检查是否有附加的命令参数，如 `--save`、`--save-dev`，以决定依赖的类型（例如：生产依赖或开发依赖）。如果没有指定，则之后会安装 `package.json` 中列出的所有依赖。

<!---->

2.  接着，`npm install` 会按优先级查找配置文件：项目级 `.npmrc` > 用户级 `.npmrc` > 全局级 `.npmrc` > npm 内置 `.npmrc`，并根据配置调整安装行为。

<!---->

3.  如果项目定义了 `preinstall` 钩子（例如：`npm run preinstall`），它会在依赖安装前被执行。可以在此步骤进行一些初始化操作，如检查版本、清理缓存等。

<!---->

4.  然后检查是否有 lock 文件，有的话会检查 package.json 中的依赖版本是否和 package-lock.json 中的依赖有冲突。如果没有冲突，直接在缓存中查找包信息。\
    如果没有 lock 文件，会先从 npm 远程仓库去获取包信息，之后根据 package.json 构建依赖树，具体过程：

- 构建依赖树时，不管其是直接依赖还是子依赖的依赖，优先将其放置在 `node_modules` 根目录。
- 当遇到相同模块时，判断已放置在依赖树的模块版本是否符合新模块的版本范围，如果符合则跳过，不符合则在当前模块的 `node_modules` 下放置该模块。

5.  之后再在缓存中依次查找依赖树的每个包：

- 不存在缓存：从 npm 远程仓库下载包，检验包的完整性，检验不通过就重新下载，检验通过会将下载的包复制到 npm 缓存目录并按照扁平化的依赖结构解压到 node-modules 中
- 存在依赖：将缓存按照扁平化的依赖结构解压到 node-modules 中

6.  生成 lock 文件

参考：

<https://juejin.cn/post/7195815771447885885?share_token=94977669-694d-4a18-b186-0969e995b6c5>

<https://juejin.cn/post/6844904022080667661#heading-49>
