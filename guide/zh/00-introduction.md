---
title: 介绍
---

### 概述

Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如工具库或应用程序。Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES6 模块可以使你自由、无缝地使用你所喜爱的工具库中那些最有用独立函数，同时不必携带其他未使用的代码。ES6 模块最终应该在所有环境下都会被原生支持，但 Rollup 可以使你现在就体验。

### 安装

```
npm install --global rollup
```

这会使得 Rollup 成为一个命令行中全局可用的命令。你也可以在项目中安装它，请看 [这一小节](#在项目中安装)。

### 快速开始

Rollup 可以通过 [命令行接口 (command line interface)](#命令行工具手册) 配合可选配置文来调用，或者可以通过 [JavaScript API](https://github.com/rollup/rollup/wiki/JavaScript-API) 来调用。运行 `rollup --help` 可以查看可用的选项和参数。

> 查看 [rollup-starter-lib](https://github.com/rollup/rollup-starter-lib) 和 [rollup-starter-app](https://github.com/rollup/rollup-starter-app) 中那些使用 Rollup 的示例类库与应用项目。

这些命令假设应用程序入口起点的名称为 `main.js`，并且最终所有导入的模块都将编译到唯一的、名为 `bundle.js` 的文件中。

对于浏览器：

```bash
# 编译为一个可以在 <script> 中使用的、自执行的函数 ('iife', Immediately Invoked Function Expression)
$ rollup main.js --file bundle.js --format iife
```

对于 Node.js:

```bash
# 编译为 CommonJS 格式的模块 ('cjs')
$ rollup main.js --file bundle.js --format cjs
```

对于浏览器和 Node.js:

```bash
# UMD 格式需要为打包产物命名
$ rollup main.js --file bundle.js --format umd --name "myBundle"
```

### 背景

通常来说，如果将您的项目分解成较小的独立部分，开发软件就会变得更容易，因为这通常会消除意料之外的交互，并大大降低您需要解决的问题的复杂性，并且首先编写较小的项目并 [不一定是最优解](https://medium.com/@Rich_Harris/small-modules-it-s-not-quite-that-simple-3ca532d65de4)。不幸的是，JavaScript 之前并未在语言中作为核心功能包含此功能。

这在 ES6 修订版的 JavaScript 中终于改变了，它包含了用于导入和导出功能和数据的语法，以便它们可以在独立脚本之间共享。虽然规范现在已经确定，但它只在现代浏览器中实现，但 Node.js 中却还并不完全支持。Rollup 允许您使用新的模块系统编写代码，然后将其编译回现有的支持的格式，例如 CommonJS 模块、AMD 模块和 IIFE 风格脚本。这意味着您可以编写 _面向未来_ 的代码，并且还可以获得模块化的巨大收益，例如更快的加载时间和更少的冗余。

### Tree-shaking

除了使用 ES 模块之外，Rollup 还会对您导入的代码进行静态分析，并删除任何实际上未使用的内容。这允许您在现有工具和模块的基础上构建，而无需增加额外的依赖项或使项目的大小膨胀。

例如，在使用 CommonJS 时，_必须导入整个工具库_。

```js
// 使用 CommonJS 整个导入了 utils 库
var utils = require( 'utils' );
var query = 'Rollup';
// 使用 utils 对象的 ajax 方法
utils.ajax(`https://api.example.com?search=${query}`).then(handleResponse);
```

但是在使用 ES 模块时，无需导入整个 `utils` 对象，我们只需导入我们所需的 `ajax` 函数：

```js
// 使用 ES 模块语法按需导入 ajax 函数
import { ajax } from 'utils';
var query = 'Rollup';
// 调用 ajax 函数
ajax(`https://api.example.com?search=${query}`).then( handleResponse );
```

正是由于 Rollup 只引入最基本最精简代码，所以可以生成轻量、快速，以及低复杂度的工具库和应用程序。因为这种基于显式的 `import` 和 `export` 语句的方式远比在编译后的输出代码中，简单地运行自动 minifier 检测未使用的变量来得更有效。

### 兼容性

#### 导入 CommonJS

Rollup 可以 [通过插件](https://github.com/rollup/rollup-plugin-commonjs) 导入目前通用的 CommonJS 模块。

#### 发布 ES 模块

为了确保你的 ES 模块可以直接在那些现在使用 CommonJS 的工具中工作，你可以使用 Rollup 编译代码为 UMD 或 CommonJS 格式，然后在 `package.json` 文件的 `main` 属性中指向当前编译的版本。如果你的 `package.json` 也具有 `module` 字段，像 Rollup 和 [Webpack 2+](https://webpack.js.org/) 这样的能够感知和处理 ES 模块的工具将会直接 [导入 ES 模块版本](https://github.com/rollup/rollup/wiki/pkg.module)。
