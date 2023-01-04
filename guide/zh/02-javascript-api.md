---
title: JavaScript API
---

Rollup 提供了一个通过 Node.js 使用的 JavaScript API。你应该很少需要使用这个，大多数情况都是使用命令行 API，除非您要扩展 Rollup 本身或将其用于某些更深奥的东西，例如自行编程生成产物。

### rollup.rollup

`rollup.rollup` 函数接收输入选项对象作为参数，并返回一个 Promise，该 Promise 最终会返回具有如下所示各种属性和方法的产物对象 `bundle`。在此步骤期间，Rollup 将构建模块图并执行除屑优化（Tree-shaking），但不会生成任何输出。

在产物对象上，您可以使用不同的输出选项对象调用 `bundle.generate` 多次，以在内存中生成不同的产物。如果您直接想将它们写入磁盘，请使用 `bundle.write`。

一旦完成产物 `bundle` 对象的打包，应调用 `bundle.close()`，这将使插件能够通过 [`closeBundle`](#closebundle) 钩子清理其他外部过程或服务。

如果在任一阶段发生错误，它将返回一个带有 Error 的中断 Promise，您可以通过它们的代号 `code` 属性识别它们。除了 `code` 和 `message` 之外，许多错误还有其他属性，您可以用于自定义错误报告，请参见 [`utils/error.ts`](https://github.com/rollup/rollup/blob/master/src/utils/error.ts)，这是一个包含所有错误和警告以及它们的代号和属性的完整列表。

```javascript
import { rollup } from 'rollup';

// 下面详细介绍了这些选项的详细信息
const inputOptions = {...};

// 您可以从相同的输入创建多个输出，
// 以生成例如 CommonJS 和 ESM 等不同格式的文件。
const outputOptionsList = [{...}, {...}];

build();

async function build() {
  let bundle;
  let buildFailed = false;
  try {
    // create a bundle
    bundle = await rollup(inputOptions);

    // an array of file names this bundle depends on
    console.log(bundle.watchFiles);

    await generateOutputs(bundle);
  } catch (error) {
    buildFailed = true;
    // do some error reporting
    console.error(error);
  }
  if (bundle) {
    // closes the bundle
    await bundle.close();
  }
  process.exit(buildFailed ? 1 : 0);
}

async function generateOutputs(bundle) {
  for (const outputOptions of outputOptionsList) {
    // generate output specific code in-memory
    // you can call this function multiple times on the same bundle object
    // replace bundle.generate with bundle.write to directly write to disk
    const { output } = await bundle.generate(outputOptions);

    for (const chunkOrAsset of output) {
      if (chunkOrAsset.type === 'asset') {
        // For assets, this contains
        // {
        //   fileName: string,              // the asset file name
        //   source: string | Uint8Array    // the asset source
        //   type: 'asset'                  // signifies that this is an asset
        // }
        console.log('Asset', chunkOrAsset);
      } else {
        // For chunks, this contains
        // {
        //   code: string,                  // the generated JS code
        //   dynamicImports: string[],      // external modules imported dynamically by the chunk
        //   exports: string[],             // exported variable names
        //   facadeModuleId: string | null, // the id of a module that this chunk corresponds to
        //   fileName: string,              // the chunk file name
        //   implicitlyLoadedBefore: string[]; // entries that should only be loaded after this chunk
        //   imports: string[],             // external modules imported statically by the chunk
        //   importedBindings: {[imported: string]: string[]} // imported bindings per dependency
        //   isDynamicEntry: boolean,       // is this chunk a dynamic entry point
        //   isEntry: boolean,              // is this chunk a static entry point
        //   isImplicitEntry: boolean,      // should this chunk only be loaded after other chunks
        //   map: string | null,            // sourcemaps if present
        //   modules: {                     // information about the modules in this chunk
        //     [id: string]: {
        //       renderedExports: string[]; // exported variable names that were included
        //       removedExports: string[];  // exported variable names that were removed
        //       renderedLength: number;    // the length of the remaining code in this module
        //       originalLength: number;    // the original length of the code in this module
        //       code: string | null;       // remaining code in this module
        //     };
        //   },
        //   name: string                   // the name of this chunk as used in naming patterns
        //   referencedFiles: string[]      // files referenced via import.meta.ROLLUP_FILE_URL_<id>
        //   type: 'chunk',                 // signifies that this is a chunk
        // }
        console.log('Chunk', chunkOrAsset.modules);
      }
    }
  }
}
```

#### inputOptions object

The `inputOptions` object can contain the following properties (see the [big list of options](guide/en/#big-list-of-options) for full details on these):

```js
const inputOptions = {
  // core input options
  external,
  input, // conditionally required
  plugins,

  // advanced input options
  cache,
  onwarn,
  preserveEntrySignatures,
  strictDeprecations,

  // danger zone
  acorn,
  acornInjectPlugins,
  context,
  moduleContext,
  preserveSymlinks,
  shimMissingExports,
  treeshake,

  // experimental
  experimentalCacheExpiry,
  perf
};
```

#### outputOptions object

The `outputOptions` object can contain the following properties (see the [big list of options](guide/en/#big-list-of-options) for full details on these):

```js
const outputOptions = {
  // core output options
  dir,
  file,
  format, // required
  globals,
  name,
  plugins,

  // advanced output options
  assetFileNames,
  banner,
  chunkFileNames,
  compact,
  entryFileNames,
  extend,
  externalLiveBindings,
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
  sourcemapExcludeSources,
  sourcemapFile,
  sourcemapPathTransform,
  validate,

  // danger zone
  amd,
  esModule,
  exports,
  freeze,
  indent,
  namespaceToStringTag,
  noConflict,
  preferConst,
  sanitizeFileName,
  strict,
  systemNullSetters
};
```

### rollup.watch

Rollup also provides a `rollup.watch` function that rebuilds your bundle when it detects that the individual modules have changed on disk. It is used internally when you run Rollup from the command line with the `--watch` flag. Note that when using watch mode via the JavaScript API, it is your responsibility to call `event.result.close()` in response to the `BUNDLE_END` event to allow plugins to clean up resources in the [`closeBundle`](guide/en/#closebundle) hook, see below.

```js
const rollup = require('rollup');

const watchOptions = {...};
const watcher = rollup.watch(watchOptions);

watcher.on('event', event => {
  // event.code can be one of:
  //   START        — the watcher is (re)starting
  //   BUNDLE_START — building an individual bundle
  //                  * event.input will be the input options object if present
  //                  * event.output contains an array of the "file" or
  //                    "dir" option values of the generated outputs
  //   BUNDLE_END   — finished building a bundle
  //                  * event.input will be the input options object if present
  //                  * event.output contains an array of the "file" or
  //                    "dir" option values of the generated outputs
  //                  * event.duration is the build duration in milliseconds
  //                  * event.result contains the bundle object that can be
  //                    used to generate additional outputs by calling
  //                    bundle.generate or bundle.write. This is especially
  //                    important when the watch.skipWrite option is used.
  //                  You should call "event.result.close()" once you are done
  //                  generating outputs, or if you do not generate outputs.
  //                  This will allow plugins to clean up resources via the
  //                  "closeBundle" hook.
  //   END          — finished building all bundles
  //   ERROR        — encountered an error while bundling
  //                  * event.error contains the error that was thrown
  //                  * event.result is null for build errors and contains the
  //                    bundle object for output generation errors. As with
  //                    "BUNDLE_END", you should call "event.result.close()" if
  //                    present once you are done.
  // If you return a Promise from your event handler, Rollup will wait until the
  // Promise is resolved before continuing.
});

// This will make sure that bundles are properly closed after each run
watcher.on('event', ({ result }) => {
  if (result) {
  	result.close();
  }
});

// Additionally, you can hook into the following. Again, return a Promise to
// make Rollup wait at that stage:
watcher.on('change', (id, { event }) => { /* a file was modified */ })
watcher.on('restart', () => { /* a new run was triggered */ })
watcher.on('close', () => { /* the watcher was closed, see below */ })

// to stop watching
watcher.close();
```

#### watchOptions

The `watchOptions` argument is a config (or an array of configs) that you would export from a config file.

```js
const watchOptions = {
  ...inputOptions,
  output: [outputOptions],
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

See above for details on `inputOptions` and `outputOptions`, or consult the [big list of options](guide/en/#big-list-of-options) for info on `chokidar`, `include` and `exclude`.

#### Programmatically loading a config file

In order to aid in generating such a config, rollup exposes the helper it uses to load config files in its command line interface via a separate entry-point. This helper receives a resolved `fileName` and optionally an object containing command line parameters:

```js
const { loadConfigFile } = require('rollup/loadConfigFile');
const path = require('node:path');
const rollup = require('rollup');

// load the config file next to the current script;
// the provided config object has the same effect as passing "--format es"
// on the command line and will override the format of all outputs
loadConfigFile(path.resolve(__dirname, 'rollup.config.js'), { format: 'es' }).then(
  async ({ options, warnings }) => {
    // "warnings" wraps the default `onwarn` handler passed by the CLI.
    // This prints all warnings up to this point:
    console.log(`We currently have ${warnings.count} warnings`);

    // This prints all deferred warnings
    warnings.flush();

    // options is an array of "inputOptions" objects with an additional "output"
    // property that contains an array of "outputOptions".
    // The following will generate all outputs for all inputs, and write them to disk the same
    // way the CLI does it:
    for (const optionsObj of options) {
      const bundle = await rollup.rollup(optionsObj);
      await Promise.all(optionsObj.output.map(bundle.write));
    }

    // You can also pass this directly to "rollup.watch"
    rollup.watch(options);
  }
);
```
