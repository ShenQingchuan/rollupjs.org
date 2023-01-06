---
title: 命令行接口
---

Rollup 通常应该通过命令行形式使用。你也可以提供一个 Rollup 配置文件，以简化命令行使用，还能启用高级 Rollup 功能。

### 配置文件

尽管配置文件是可选的，但使用配置文件能更方便地利用 Rollup 的丰富功能，因此我们 **推荐** 你这样用。配置文件同样是一个 ES 模块，它应该导出一个默认对象，其中包含所需的选项：

```javascript
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};
```

通常，它的文件名应该是 `rollup.config.js` 或者 `rollup.config.mjs` 并位于你项目的根目录。除非你使用了 [`--configPlugin`](guide/zh/#--configplugin-plugin) 或者 [`--bundleConfigAsCjs`](guide/zh/#--bundleconfigascjs) 选项，Rollup 都会直接使用 Node 来引入该文件。有一些 [使用原生 Node ES 模块时的注意事项](guide/zh/#使用原生-node-es-模块时的注意事项)，因为 Rollup 会遵循 [Node ESM 的语义](https://nodejs.org/docs/latest-v14.x/api/packages.html#packages_determining_module_system)。

如果你想使用 `require` 和 `module.exports` 将你的配置写成一个 CommonJS 模块，你应该将文件扩展名改为 `.cjs`。

你也可以使用其他语言来作为你的配置文件，比如 TypeScript。若你要这么做，需要安装一个相应的 Rollup 插件，比如 `@rollup/plugin-typescript`，并使用 [`--configPlugin`](guide/zh/#--configplugin-plugin) 选项：

```
rollup --config rollup.config.ts --configPlugin typescript
```

使用 `--configPlugin` 选项将总是强制将你的配置文件首先转译为 CommonJS。此外，还可以查看 [配置智能提示](guide/zh/#配置智能提示)，以了解更多在配置文件中使用 TypeScript 类型的方法。

配置文件支持下面列出的这些选项。有关每个选项的详细信息，请参阅 [选项大列表](guide/zh/#选项大列表)：

```javascript
// rollup.config.js

// 可以是一个数组（用于多个输入）
export default {
  // 输入配置的核心选项
  external,
  input, // 必需，但包含多种情景
  plugins,

  // 输入配置的进阶选项
  cache,
  onwarn,
  preserveEntrySignatures,
  strictDeprecations,

  // 需要谨慎设置的选项
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // 实验性
  experimentalCacheExpiry,
  perf,

  // 必需（可以是数组，用于多个输出）
  output: {
    // 输出配置的核心选项
    dir,
    file,
    format, // 必需
    globals,
    name,
    plugins,

    // 输出配置的进阶选项
    assetFileNames,
    banner,
    chunkFileNames,
    compact,
    entryFileNames,
    extend,
    footer,
    hoistTransitiveImports,
    inlineDynamicImports,
    interop,
    intro,
    manualChunks,
    minifyInternalExports,
    outro,
    paths,
    preserveModules,
    preserveModulesRoot,
    sourcemap,
    sourcemapBaseUrl,
    sourcemapExcludeSources,
    sourcemapFile,
    sourcemapPathTransform,
    validate,

    // 需要谨慎设置的选项
    amd,
    esModule,
    exports,
    externalLiveBindings,
    freeze,
    indent,
    namespaceToStringTag,
    noConflict,
    preferConst,
    sanitizeFileName,
    strict,
    systemNullSetters
  },

  watch: {
    buildDelay,
    chokidar,
    clearScreen,
    skipWrite,
    exclude,
    include
  }
};
```

你可以从你的配置文件中导出一个 **数组**，以一次性构建来自几个输入来源不相关的产物，甚至在 watch 模式下也可以。若要用相同的输入构建不同的产物，你需要为每个输入提供一个输出选项数组：

```javascript
// rollup.config.js（打包多个不同的产物）

export default [
  {
    input: 'main-a.js',
    output: {
      file: 'dist/bundle-a.js',
      format: 'cjs'
    }
  },
  {
    input: 'main-b.js',
    output: [
      {
        file: 'dist/bundle-b1.js',
        format: 'cjs'
      },
      {
        file: 'dist/bundle-b2.js',
        format: 'es'
      }
    ]
  }
];
```

如果你想要异步创建你的配置，Rollup 也可以处理一个 `Promise`，它最终会获得一个对象或者一个数组。

```javascript
// rollup.config.js
import fetch from 'node-fetch';
export default fetch('/some-remote-service-or-file-which-returns-actual-config');
```

同样，你也可以这样做：

```javascript
// rollup.config.js（最终获得一个数组的 Promise）
export default Promise.all([fetch('get-config-1'), fetch('get-config-2')]);
```

（在命令行形式中）使用配置文件来使用 Rollup，传递 `--config` 或 `-c` 标志：

```
# 将自定义配置文件的所在位置传递给 Rollup
rollup --config my.config.js

# 如果你没有传递文件名，
# Rollup 将按照以下顺序尝试加载配置文件：
# rollup.config.mjs -> rollup.config.cjs -> rollup.config.js
rollup --config
```

你还可以导出一个返回上述配置格式之一的函数。 该函数将接收当前命令行参数，以便其可以根据 [`--silent`](guide/zh/#--silent) 这样的选项动态调整你的配置。如果你以 `config` 为前缀，你甚至可以定义自己的命令行选项：

```javascript
// rollup.config.js
import defaultConfig from './rollup.default.config.js';
import debugConfig from './rollup.debug.config.js';

export default commandLineArgs => {
  if (commandLineArgs.configDebug === true) {
    return debugConfig;
  }
  return defaultConfig;
};
```

（有了上述配置定义）如果现在运行 `rollup --config --configDebug`，将使用 `debugConfig`。

默认情况下，命令行参数将始终覆盖来自配置文件的相应值。如果要更改此行为逻辑，则可以通过从 `commandLineArgs` 对象中删除它们来使 Rollup 忽略命令行参数：

```javascript
// rollup.config.js
export default commandLineArgs => {
  const inputBase = commandLineArgs.input || 'main.js';

  // 这将使 Rollup 忽略 CLI 参数
  delete commandLineArgs.input;
  return {
    input: 'src/entries/' + inputBase,
    output: {...}
  }
}
```

#### 配置智能提示

由于 Rollup 同时发布了 TypeScript 类型，因此可以使用 JSDoc 类型提示利用 IDE 的 Intellisense：

```javascript
// rollup.config.js
/**
 * @type {import('rollup').RollupOptions}
 */
const config = {
  /* 你的配置 */
};
export default config;
```

或者，你可以使用 defineConfig 帮助函数，它可以在你书写参数时自动带有智能提示，而无需 JSDoc 注释：

```javascript
// rollup.config.js
import { defineConfig } from 'rollup';

export default defineConfig({
  /* 你的配置 */
});
```

除了 `RollupOptions` 和封装此类型的 `defineConfig` 帮助函数之外，以下类型也可能派上用场：

- `OutputOptions`：配置文件的 `output` 部分
- `Plugin`：插件对象，提供了一个 `name` 和一些钩子。所有的钩子都是完全标注了类型的，可以帮助插件的开发。
- `PluginImpl`：将选项对象映射到插件对象的函数。 大多数公共 Rollup 插件都遵循此模式。

您还可以通过 [`--configPlugin`](guide/zh/#--configplugin-plugin) 选项直接在 TypeScript 中编写配置。使用 TypeScript，您可以直接导入 RollupOptions 类型：

```typescript
import type { RollupOptions } from 'rollup';

const config: RollupOptions = {
  /* 你的配置 */
};
export default config;
```

### 与 JavaScript API 的差异

虽然配置文件提供了一种简单的方法来配置 Rollup，但它们也限制了 Rollup 如何调用和配置。特别是如果你要将 Rollup 这个工具打包到另一个构建工具中、或者想将其集成到更复杂的构建过程中，直接从你的脚本程序调用 Rollup 可能会更好。

如果你想从配置文件切换为使用 [JavaScript API](guide/zh/#javascript-api)，需要注意以下几个重要的差异：

- 使用 JavaScript API 时，传递给 `rollup.rollup` 的配置必须是一个对象，并且不能被包在一个 Promise 或函数中。
- 你不能再直接使用数组形式的配置了，而是需要对每组 `inputOptions` 运行一次 `rollup.rollup`。
- `output` 选项将被忽略。你应该对每组 `outputOptions` 运行一次 `bundle.generate(outputOptions)` 或 `bundle.write(outputOptions)`。

### 从 Node 包加载配置

为了实现互操作性，Rollup 也支持从安装到 `node_modules` 的包中加载配置文件：

```
# 这将首先尝试加载 "rollup-config-my-special-config" 这个包；
# 如果失败，它将尝试加载 "my-special-config"
rollup --config node:my-special-config
```

### 使用原生 Node ES 模块时的注意事项

尤其是在从旧版 Rollup 升级时，使用原生 ES 模块作为配置文件时，需要注意一些事项。

#### 获取当前目录

使用 CommonJS 文件时，人们常常使用 `__dirname` 访问当前目录并将相对路径解析为绝对路径。这在原生 ES 模块中不受支持。相反，我们建议以下方法来为外部模块生成一个绝对定位的标识符：

```js
// rollup.config.js
import { fileURLToPath } from 'node:url'

export default {
  ...,
  // 为 <currentdir>/src/some-external-file.js 生成绝对路径
  external: [fileURLToPath(new URL('src/some-external-file.js', import.meta.url))]
};
```

#### 导入 package.json

有时候导入 package.json 文件会有用，例如自动将你的依赖项标记为“外部”。根据你的 Node 版本，有不同的方法可以做到这一点：

- 对于 Node 17.5+，你可以使用导入断言

  ```js
  import pkg from './package.json' assert { type: 'json' };

  export default {
    // Mark package dependencies as "external". Rest of configuration omitted.
    external: Object.keys(pkg.dependencies)
  };
  ```

- 对于更早的 Node 版本，你可以使用 `createRequire`

  ```js
  import { createRequire } from 'node:module';
  const require = createRequire(import.meta.url);
  const pkg = require('./package.json');

  // ...
  ```

- 或者也可以直接从磁盘读取和解析该文件

  ```js
  // rollup.config.mjs
  import { readFileSync } from 'node:fs';

  // 使用 import.meta.url 来创建配置文件相对于 package.json 的路径，而避免使用 process.cwd()
  // 对此有疑惑可参阅：https://nodejs.org/docs/latest-v16.x/api/esm.html#importmetaurl
  const packageJson = JSON.parse(readFileSync(new URL('./package.json', import.meta.url)));

  // ...
  ```

### 命令行选项

许多配置文件中的选项都有等价的命令行选项。在这些情况下，在此处传递的任何参数都将覆盖已经被使用的配置文件。下面是所有支持的选项的列表：

```text
-c, --config <filename>     使用此配置文件（如果使用了该选项但未指定值，则默认为 rollup.config.js）
-d, --dir <dirname>         生成产物分块的目标文件夹（如果不存在，则输出到 stdout）
-e, --external <ids>        要排除的模块 ID 列表，以逗号分隔
-f, --format <format>       输出类型（amd、cjs、es、iife、umd、system）
-g, --globals <pairs>       `moduleID:Global` 键值对的列表，以逗号分隔
-h, --help                  显示命令行帮助信息
-i, --input <filename>      输入来源（<entry file> 的替代）
-m, --sourcemap             生成源映射（使用 `-m inline` 便是内嵌映射）（inline sourcemap）
-n, --name <name>           UMD 导出的名称
-o, --file <output>         单个输出文件（如果不存在，则输出到 stdout
-p, --plugin <plugin>       使用指定的插件（可使用多个该选项）
-v, --version               显示 Rollup 版本号
-w, --watch                 监听需打包的文件并在更改时重新构建
--amd.autoId                根据块名自动生成 AMD ID（默认是匿名的）
--amd.basePath <prefix>     在自动生成的 AMD ID 之前添加的路径
--amd.define <name>         代替 `define` 的函数
--amd.forceJsExtensionForImports 在 AMD 导入中强制使用 .js 扩展名
--assetFileNames <pattern>  打包出的的资源文件的名字格式
--banner <text>             在产物顶部插入的代码（外部包装器外）
--chunkFileNames <pattern>  打包出的次级产物切块的名称模式
--compact                   是否压缩包装器代码
--context <variable>        指定顶级 this 值
--no-dynamicImportInCjs     将外部动态 CommonJS 导入写为 require
--entryFileNames <pattern>  产物中作为入口的切块的命名模式
--environment <values>      传递到配置文件的设置（参见示例）
--no-esModule               不添加 __esModule 属性
--exports <mode>            指定导出模式（auto、default、named、none）
--extend                    通过 `--name` 定义，拓展全局变量
--no-externalImportAssertions 在 "es" 输出产物中忽略导入断言
--no-externalLiveBindings   不生成实时绑定（Live bindings）的代码
--failAfterWarnings         只要构建生成了警告，就退出并显示错误，
--footer <text>             在产物结尾插入代码（在包装器代码之外）
--no-freeze                 不要冻结命名空间对象
--generatedCode <preset>    使用的代码功能（es5/es2015）
--generatedCode.arrowFunctions 在生成代码中使用箭头函数
--generatedCode.constBindings 在生成代码中使用 "const"
--generatedCode.objectShorthand 在生成代码中使用属性缩写
--no-generatedCode.reservedNamesAsProps 始终在保留字作为属性名时加引号
--generatedCode.symbols     在生成代码中 symbol
--no-hoistTransitiveImports 不提升传递性的导入到产物的入口块
--no-indent                 结果中不进行缩进
--inlineDynamicImports      使用动态导入时创建单独的产物
--no-interop                不要包含互操作块
--intro <text>              在产物顶部插入代码（在包装器代码内）
--no-makeAbsoluteExternalsRelative 防止规范化过程（normalization）作用于外部导入上
--maxParallelFileOps <value> 并行读入文件的最大值
--minifyInternalExports     强制开启或禁用内部导出的最小化压缩
--noConflict                Generate a noConflict method for UMD globals
--outro <text>              Code to insert at end of bundle (inside wrapper)
--perf                      Display performance timings
--no-preserveEntrySignatures Avoid facade chunks for entry points
--preserveModules           Preserve module structure
--preserveModulesRoot       Put preserved modules under this path at root level
--preserveSymlinks          Do not follow symlinks when resolving files
--no-sanitizeFileName       Do not replace invalid characters in file names
--shimMissingExports        Create shim variables for missing exports
--silent                    Don't print warnings
--sourcemapBaseUrl <url>    Emit absolute sourcemap URLs with given base
--sourcemapExcludeSources   Do not include source code in source maps
--sourcemapFile <file>      Specify bundle position for source maps
--stdin=ext                 Specify file extension used for stdin input
--no-stdin                  Do not read "-" from stdin
--no-strict                 Don't emit `"use strict";` in the generated modules
--strictDeprecations        Throw errors for deprecated features
--no-systemNullSetters      Do not replace empty SystemJS setters with `null`
--no-treeshake              Disable tree-shaking optimisations
--no-treeshake.annotations  Ignore pure call annotations
--treeshake.correctVarValueBeforeDeclaration Deoptimize variables until declared
--treeshake.manualPureFunctions <names> Manually declare functions as pure
--no-treeshake.moduleSideEffects Assume modules have no side effects
--no-treeshake.propertyReadSideEffects Ignore property access side effects
--no-treeshake.tryCatchDeoptimization Do not turn off try-catch-tree-shaking
--no-treeshake.unknownGlobalSideEffects Assume unknown globals do not throw
--validate                  Validate output
--waitForBundleInput        Wait for bundle input files
--watch.buildDelay <number> Throttle watch rebuilds
--no-watch.clearScreen      Do not clear the screen when rebuilding
--watch.exclude <files>     Exclude files from being watched
--watch.include <files>     Limit watching to specified files
--watch.onBundleEnd <cmd>   Shell command to run on `"BUNDLE_END"` event
--watch.onBundleStart <cmd> Shell command to run on `"BUNDLE_START"` event
--watch.onEnd <cmd>         Shell command to run on `"END"` event
--watch.onError <cmd>       Shell command to run on `"ERROR"` event
--watch.onStart <cmd>       Shell command to run on `"START"` event
--watch.skipWrite           Do not write files to disk when watching
```

The flags listed below are only available via the command line interface. All other flags correspond to and override their config file equivalents, see the [big list of options](guide/zh/#big-list-of-options) for details.

#### `-h`/`--help`

Print the help document.

#### `-p <plugin>`, `--plugin <plugin>`

Use the specified plugin. There are several ways to specify plugins here:

- Via a relative path:

  ```
  rollup -i input.js -f es -p ./my-plugin.js
  ```

  The file should export a function returning a plugin object.

- Via the name of a plugin that is installed in a local or global `node_modules` folder:

  ```
  rollup -i input.js -f es -p @rollup/plugin-node-resolve
  ```

  If the plugin name does not start with `rollup-plugin-` or `@rollup/plugin-`, Rollup will automatically try adding these prefixes:

  ```
  rollup -i input.js -f es -p node-resolve
  ```

- Via an inline implementation:

  ```
  rollup -i input.js -f es -p '{transform: (c, i) => `/* ${JSON.stringify(i)} */\n${c}`}'
  ```

If you want to load more than one plugin, you can repeat the option or supply a comma-separated list of names:

```
rollup -i input.js -f es -p node-resolve -p commonjs,json
```

By default, plugin functions will be called with no argument to create the plugin. You can however pass a custom argument as well:

```
rollup -i input.js -f es -p 'terser={output: {beautify: true, indent_level: 2}}'
```

#### `--configPlugin <plugin>`

Allows specifying Rollup plugins to transpile or otherwise control the parsing of your configuration file. The main benefit is that it allows you to use non-JavaScript configuration files. For instance the following will allow you to write your configuration in TypeScript, provided you have `@rollup/plugin-typescript` installed:

```
rollup --config rollup.config.ts --configPlugin @rollup/plugin-typescript
```

Note for Typescript: make sure you have the Rollup config file in your `tsconfig.json`'s `include` paths. For example:

```
"include": ["src/**/*", "rollup.config.ts"],
```

This option supports the same syntax as the [`--plugin`](guide/zh/#-p-plugin---plugin-plugin) option i.e., you can specify the option multiple times, you can omit the `@rollup/plugin-` prefix and just write `typescript` and you can specify plugin options via `={...}`.

Using this option will make Rollup transpile your configuration file to an ES module first before executing it. To transpile to CommonJS instead, also pass the [`--bundleConfigAsCjs`](guide/zh/#--bundleconfigascjs) option.

#### `--bundleConfigAsCjs`

This option will force your configuration to be transpiled to CommonJS.

This allows you to use CommonJS idioms like `__dirname` or `require.resolve` in your configuration even if the configuration itself is written as an ES module.

#### `-v`/`--version`

Print the installed version number.

#### `-w`/`--watch`

Rebuild the bundle when its source files change on disk.

_Note: While in watch mode, the `ROLLUP_WATCH` environment variable will be set to `"true"` by Rollup's command line interface and can be checked by other processes. Plugins should instead check [`this.meta.watchMode`](guide/zh/#thismeta), which is independent of the command line interface._

#### `--silent`

Don't print warnings to the console. If your configuration file contains an `onwarn` handler, this handler will still be called. To manually prevent that, you can access the command line options in your configuration file as described at the end of [Configuration Files](guide/zh/#configuration-files).

#### `--failAfterWarnings`

Exit the build with an error if any warnings occurred, once the build is complete.

#### `--environment <values>`

Pass additional settings to the config file via `process.ENV`.

```sh
rollup -c --environment INCLUDE_DEPS,BUILD:production
```

will set `process.env.INCLUDE_DEPS === 'true'` and `process.env.BUILD === 'production'`. You can use this option several times. In that case, subsequently set variables will overwrite previous definitions. This enables you for instance to overwrite environment variables in `package.json` scripts:

```json
{
  "scripts": {
    "build": "rollup -c --environment INCLUDE_DEPS,BUILD:production"
  }
}
```

If you call this script via:

```
npm run build -- --environment BUILD:development
```

then the config file will receive `process.env.INCLUDE_DEPS === 'true'` and `process.env.BUILD === 'development'`.

#### `--waitForBundleInput`

This will not throw an error if one of the entry point files is not available. Instead, it will wait until all files are present before starting the build. This is useful, especially in watch mode, when Rollup is consuming the output of another process.

#### `--stdin=ext`

Specify a virtual file extension when reading content from stdin. By default, Rollup will use the virtual file name `-` without an extension for content read from stdin. Some plugins, however, rely on file extensions to determine if they should process a file. See also [Reading a file from stdin](guide/zh/#reading-a-file-from-stdin).

#### `--no-stdin`

Do not read files from `stdin`. Setting this flag will prevent piping content to Rollup and make sure Rollup interprets `-` and `-.[ext]` as a regular file names instead of interpreting these as the name of `stdin`. See also [Reading a file from stdin](guide/zh/#reading-a-file-from-stdin).

#### `--watch.onStart <cmd>`, `--watch.onBundleStart <cmd>`, `--watch.onBundleEnd <cmd>`, `--watch.onEnd <cmd>`, `--watch.onError <cmd>`

When in watch mode, run a shell command `<cmd>` for a watch event code. See also [rollup.watch](guide/zh/#rollupwatch).

```sh
rollup -c --watch --watch.onEnd="node ./afterBuildScript.js"
```

### Reading a file from stdin

When using the command line interface, Rollup can also read content from stdin:

```
echo "export const foo = 42;" | rollup --format cjs --file out.js
```

When this file contains imports, Rollup will try to resolve them relative to the current working directory. When using a config file, Rollup will only use `stdin` as an entry point if the file name of the entry point is `-`. To read a non-entry-point file from stdin, just call it `-`, which is the file name that is used internally to reference `stdin`. I.e.

```js
import foo from '-';
```

in any file will prompt Rollup to try to read the imported file from `stdin` and assign the default export to `foo`. You can pass the [`--no-stdin`](guide/zh/#--no-stdin) CLI flag to Rollup to treat `-` as a regular file name instead.

As some plugins rely on file extensions to process files, you can specify a file extension for stdin via `--stdin=ext` where `ext` is the desired extension. In that case, the virtual file name will be `-.ext`:

```
echo '{"foo": 42, "bar": "ok"}' | rollup --stdin=json -p json
```

The JavaScript API will always treat `-` and `-.ext` as regular file names.
