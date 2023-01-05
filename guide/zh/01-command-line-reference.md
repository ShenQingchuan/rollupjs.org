---
title: 命令行接口
---

Rollup 通常应该通过命令行形式使用。您也可以提供一个 Rollup 配置文件，以简化命令行使用，还能启用高级 Rollup 功能。

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

通常，它的文件名应该是 `rollup.config.js` 或者 `rollup.config.mjs` 并位于你项目的根目录。除非你使用了 [`--configPlugin`](guide/en/#--configplugin-plugin) 或者 [`--bundleConfigAsCjs`](guide/en/#--bundleconfigascjs) 选项，Rollup 都会直接使用 Node 来引入该文件。有一些 [使用原生 ES 模块时的注意事项](#使用原生_ES_模块的注意事项)，因为 Rollup 会遵循 [Node ESM 的语义](https://nodejs.org/docs/latest-v14.x/api/packages.html#packages_determining_module_system)。

如果你想使用 `require` 和 `module.exports` 将你的配置写成一个 CommonJS 模块，你应该将文件扩展名改为 `.cjs`。

你也可以使用其他语言来作为你的配置文件，比如 TypeScript。若你要这么做，需要安装一个相应的 Rollup 插件，比如 `@rollup/plugin-typescript`，并使用 [`--configPlugin`](guide/en/#--configplugin-plugin) 选项：

```
rollup --config rollup.config.ts --configPlugin typescript
```

使用 `--configPlugin` 选项将总是强制将您的配置文件首先转译为 CommonJS。此外，还可以查看 [配置智能提示](#配置智能提示)，以了解更多在配置文件中使用 TypeScript 类型的方法。

配置文件支持下面列出的这些选项。有关每个选项的详细信息，请参阅 [选项大列表](#选项大列表)：

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

# 如果您没有传递文件名，
# Rollup 将按照以下顺序尝试加载配置文件：
# rollup.config.mjs -> rollup.config.cjs -> rollup.config.js
rollup --config
```

You can also export a function that returns any of the above configuration formats. This function will be passed the current command line arguments so that you can dynamically adapt your configuration to respect e.g. [`--silent`](guide/en/#--silent). You can even define your own command line options if you prefix them with `config`:

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

If you now run `rollup --config --configDebug`, the debug configuration will be used.

By default, command line arguments will always override the respective values exported from a config file. If you want to change this behaviour, you can make Rollup ignore command line arguments by deleting them from the `commandLineArgs` object:

```javascript
// rollup.config.js
export default commandLineArgs => {
  const inputBase = commandLineArgs.input || 'main.js';

  // this will make Rollup ignore the CLI argument
  delete commandLineArgs.input;
  return {
    input: 'src/entries/' + inputBase,
    output: {...}
  }
}
```

#### Config Intellisense

Since Rollup ships with TypeScript typings, you can leverage your IDE's Intellisense with JSDoc type hints:

```javascript
// rollup.config.js
/**
 * @type {import('rollup').RollupOptions}
 */
const config = {
  /* your config */
};
export default config;
```

Alternatively you can use the `defineConfig` helper, which should provide Intellisense without the need for JSDoc annotations:

```javascript
// rollup.config.js
import { defineConfig } from 'rollup';

export default defineConfig({
  /* your config */
});
```

Besides `RollupOptions` and the `defineConfig` helper that encapsulates this type, the following types can prove useful as well:

- `OutputOptions`: The `output` part of a config file
- `Plugin`: A plugin object that provides a `name` and some hooks. All hooks are fully typed to aid in plugin development.
- `PluginImpl`: A function that maps an options object to a plugin object. Most public Rollup plugins follow this pattern.

You can also directly write your config in TypeScript via the [`--configPlugin`](guide/en/#--configplugin-plugin) option. With TypeScript, you can import the `RollupOptions` type directly:

```typescript
import type { RollupOptions } from 'rollup';

const config: RollupOptions = {
  /* your config */
};
export default config;
```

### Differences to the JavaScript API

While config files provide an easy way to configure Rollup, they also limit how Rollup can be invoked and configured. Especially if you are bundling Rollup into another build tool or want to integrate it into an advanced build process, it may be better to directly invoke Rollup programmatically from your scripts.

If you want to switch from config files to using the [JavaScript API](guide/en/#javascript-api) at some point, there are some important differences to be aware of:

- When using the JavaScript API, the configuration passed to `rollup.rollup` must be an object and cannot be wrapped in a Promise or a function.
- You can no longer use an array of configurations. Instead, you should run `rollup.rollup` once for each set of `inputOptions`.
- The `output` option will be ignored. Instead, you should run `bundle.generate(outputOptions)` or `bundle.write(outputOptions)` once for each set of `outputOptions`.

### Loading a configuration from a Node package

For interoperability, Rollup also supports loading configuration files from packages installed into `node_modules`:

```
# this will first try to load the package "rollup-config-my-special-config";
# if that fails, it will then try to load "my-special-config"
rollup --config node:my-special-config
```

### Caveats when using native Node ES modules

Especially when upgrading from an older Rollup version, there are some things you need to be aware of when using a native ES module for your configuration file.

#### Getting the current directory

With CommonJS files, people often use `__dirname` to access the current directory and resolve relative paths to absolute paths. This is not supported for native ES modules. Instead, we recommend the following approach e.g. to generate an absolute id for an external module:

```js
// rollup.config.js
import { fileURLToPath } from 'node:url'

export default {
  ...,
  // generates an absolute path for <currentdir>/src/some-external-file.js
  external: [fileURLToPath(new URL('src/some-external-file.js', import.meta.url))]
};
```

#### Importing package.json

It can be useful to import your package file to e.g. mark your dependencies as "external" automatically. Depending on your Node version, there are different ways of doing that:

- For Node 17.5+, you can use an import assertion

  ```js
  import pkg from './package.json' assert { type: 'json' };

  export default {
    // Mark package dependencies as "external". Rest of configuration omitted.
    external: Object.keys(pkg.dependencies)
  };
  ```

- For older Node versions, you can use `createRequire`

  ```js
  import { createRequire } from 'node:module';
  const require = createRequire(import.meta.url);
  const pkg = require('./package.json');

  // ...
  ```

- Or just directly read and parse the file from disk

  ```js
  // rollup.config.mjs
  import { readFileSync } from 'node:fs';

  // Use import.meta.url to make the path relative to the current source file instead of process.cwd()
  // For more info: https://nodejs.org/docs/latest-v16.x/api/esm.html#importmetaurl
  const packageJson = JSON.parse(readFileSync(new URL('./package.json', import.meta.url)));

  // ...
  ```

### Command line flags

Many options have command line equivalents. In those cases, any arguments passed here will override the config file, if you're using one. This is a list of all supported options:

```text
-c, --config <filename>     Use this config file (if argument is used but value
                              is unspecified, defaults to rollup.config.js)
-d, --dir <dirname>         Directory for chunks (if absent, prints to stdout)
-e, --external <ids>        Comma-separate list of module IDs to exclude
-f, --format <format>       Type of output (amd, cjs, es, iife, umd, system)
-g, --globals <pairs>       Comma-separate list of `moduleID:Global` pairs
-h, --help                  Show this help message
-i, --input <filename>      Input (alternative to <entry file>)
-m, --sourcemap             Generate sourcemap (`-m inline` for inline map)
-n, --name <name>           Name for UMD export
-o, --file <output>         Single output file (if absent, prints to stdout)
-p, --plugin <plugin>       Use the plugin specified (may be repeated)
-v, --version               Show version number
-w, --watch                 Watch files in bundle and rebuild on changes
--amd.autoId                Generate the AMD ID based off the chunk name
--amd.basePath <prefix>     Path to prepend to auto generated AMD ID
--amd.define <name>         Function to use in place of `define`
--amd.forceJsExtensionForImports Use `.js` extension in AMD imports
--amd.id <id>               ID for AMD module (default is anonymous)
--assetFileNames <pattern>  Name pattern for emitted assets
--banner <text>             Code to insert at top of bundle (outside wrapper)
--chunkFileNames <pattern>  Name pattern for emitted secondary chunks
--compact                   Minify wrapper code
--context <variable>        Specify top-level `this` value
--no-dynamicImportInCjs     Write external dynamic CommonJS imports as require
--entryFileNames <pattern>  Name pattern for emitted entry chunks
--environment <values>      Settings passed to config file (see example)
--no-esModule               Do not add __esModule property
--exports <mode>            Specify export mode (auto, default, named, none)
--extend                    Extend global variable defined by --name
--no-externalImportAssertions Omit import assertions in "es" output
--no-externalLiveBindings   Do not generate code to support live bindings
--failAfterWarnings         Exit with an error if the build produced warnings
--footer <text>             Code to insert at end of bundle (outside wrapper)
--no-freeze                 Do not freeze namespace objects
--generatedCode <preset>    Which code features to use (es5/es2015)
--generatedCode.arrowFunctions Use arrow functions in generated code
--generatedCode.constBindings Use "const" in generated code
--generatedCode.objectShorthand Use shorthand properties in generated code
--no-generatedCode.reservedNamesAsProps Always quote reserved names as props
--generatedCode.symbols     Use symbols in generated code
--no-hoistTransitiveImports Do not hoist transitive imports into entry chunks
--no-indent                 Don't indent result
--inlineDynamicImports      Create single bundle when using dynamic imports
--no-interop                Do not include interop block
--intro <text>              Code to insert at top of bundle (inside wrapper)
--no-makeAbsoluteExternalsRelative Prevent normalization of external imports
--maxParallelFileOps <value> How many files to read in parallel
--minifyInternalExports     Force or disable minification of internal exports
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

The flags listed below are only available via the command line interface. All other flags correspond to and override their config file equivalents, see the [big list of options](guide/en/#big-list-of-options) for details.

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

This option supports the same syntax as the [`--plugin`](guide/en/#-p-plugin---plugin-plugin) option i.e., you can specify the option multiple times, you can omit the `@rollup/plugin-` prefix and just write `typescript` and you can specify plugin options via `={...}`.

Using this option will make Rollup transpile your configuration file to an ES module first before executing it. To transpile to CommonJS instead, also pass the [`--bundleConfigAsCjs`](guide/en/#--bundleconfigascjs) option.

#### `--bundleConfigAsCjs`

This option will force your configuration to be transpiled to CommonJS.

This allows you to use CommonJS idioms like `__dirname` or `require.resolve` in your configuration even if the configuration itself is written as an ES module.

#### `-v`/`--version`

Print the installed version number.

#### `-w`/`--watch`

Rebuild the bundle when its source files change on disk.

_Note: While in watch mode, the `ROLLUP_WATCH` environment variable will be set to `"true"` by Rollup's command line interface and can be checked by other processes. Plugins should instead check [`this.meta.watchMode`](guide/en/#thismeta), which is independent of the command line interface._

#### `--silent`

Don't print warnings to the console. If your configuration file contains an `onwarn` handler, this handler will still be called. To manually prevent that, you can access the command line options in your configuration file as described at the end of [Configuration Files](guide/en/#configuration-files).

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

Specify a virtual file extension when reading content from stdin. By default, Rollup will use the virtual file name `-` without an extension for content read from stdin. Some plugins, however, rely on file extensions to determine if they should process a file. See also [Reading a file from stdin](guide/en/#reading-a-file-from-stdin).

#### `--no-stdin`

Do not read files from `stdin`. Setting this flag will prevent piping content to Rollup and make sure Rollup interprets `-` and `-.[ext]` as a regular file names instead of interpreting these as the name of `stdin`. See also [Reading a file from stdin](guide/en/#reading-a-file-from-stdin).

#### `--watch.onStart <cmd>`, `--watch.onBundleStart <cmd>`, `--watch.onBundleEnd <cmd>`, `--watch.onEnd <cmd>`, `--watch.onError <cmd>`

When in watch mode, run a shell command `<cmd>` for a watch event code. See also [rollup.watch](guide/en/#rollupwatch).

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

in any file will prompt Rollup to try to read the imported file from `stdin` and assign the default export to `foo`. You can pass the [`--no-stdin`](guide/en/#--no-stdin) CLI flag to Rollup to treat `-` as a regular file name instead.

As some plugins rely on file extensions to process files, you can specify a file extension for stdin via `--stdin=ext` where `ext` is the desired extension. In that case, the virtual file name will be `-.ext`:

```
echo '{"foo": 42, "bar": "ok"}' | rollup --stdin=json -p json
```

The JavaScript API will always treat `-` and `-.ext` as regular file names.
