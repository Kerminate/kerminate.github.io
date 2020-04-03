---
title: Webpack5 changelog
date: 2020-03-27 10:02:48
tags:
---

# 总体方向

这个版本重点关注以下内容：

- 我们尝试通过持久性缓存提高构建性能。
- 我们尝试通过更好的算法和默认值来改善长期缓存。
- 我们尝试使用更好的 Tree Shaking 和代码生成来改善打包体积。
- 我们尝试清除处于怪异状态的内部结构，同时在 v4 中不引入任何重大改变的情况下实现功能。
- 我们现在尝试通过引入重大改变来为将来的功能做准备，以使我们能够尽可能地长时间使用 v5。

<!-- more -->

# 迁移指导

=> [点击此处查看迁移指导](https://github.com/webpack/changelog-v5/blob/master/MIGRATION%20GUIDE.md) <=

# 主要变化

## 删除过时的内容

v4 中弃用的内容均已删除。迁移到 v5 时，请确保您的 webpack4 版本在构建时不会打印出弃用警告。

以下是一些已删除但在 v4 中没有弃用警告的内容：

- IgnorePlugin and BannerPlugin must now be passed an options object.

## 弃用代码

新的弃用包括弃用代码，因此更易于引用。（从 beta.7 开始）

## 语法弃用

`require.include` 已被弃用，使用时默认会发出警告。

可以使用 `Rule.parser.requireInclude` 将以上语法行为更改为允许，但是不建议使用或禁用。（从 beta.1 开始）

# 自动删除 Node.js polyfills

早期，webpack 的目标是允许在浏览器中运行大多数 node.js 模块，但是模块的环境发生了变化，许多模块现在主要是为前端而编写的。 webpack <= 4 的版本附带了许多 node.js 核心模块的 polyfills，一旦模块使用任何核心模块（比如 crypto 模块），该模块就会自动应用。

尽管这使得为 node.js 编写模块变得容易，但它会将这些巨大的 polyfills 添加到包中。在很多情况下，这些 polyfills 是不必要的。

webpack5 不再自动添加这些核心模块，并专注于与前端兼容的模块。

兼容：

- 尽可能尝试使用与前端兼容的模块。
- 可以为 node.js 的核心模块手动添加 polyfill。错误消息会提示你如何去实现它。
- 包作者：使用 `package.json` 中的 `browser` 字段使包与前端兼容，为浏览器提高可替代的实现或依赖。

## 固定的 chunk 和模块 IDs

webpack5 添加了适合长期缓存的新算法，在生产环境中会默认启用这些功能。

`chunkIds: "deterministic", moduleIds: "deterministic"`

该算法以确定性方式将简短的（3个或4个字符）数字 ID 分配给模块和 chunks。
这是包大小和长期缓存之间的折衷方案。

`moduleIds/chunkIds: false` 禁用默认行为，并且可以通过插件提供自定义算法。请注意，在 webpack4 中，没有提供自定义插件时的 `moduleIds / chunkIds: false` 构建正常，而在 webpack5 中，您必须提供自定义插件。

迁移：最好使用 `chunkIds` 和 `moduleIds` 的默认值。 您还可以选择使用旧的默认设置`chunkIds: "size", moduleIds: "size"`，这将生成较小的包，但可能常常会无法有效进行缓存。

注意：在 webpack4 中，`moduleIds` 配置为 `hashed` 会导致 gzip 性能降低。这与更改的模块顺序有关，并且已得到修复。 （从 beta.1 开始）

## 固定的混淆导出名字

添加了新算法来处理导出名字，默认情况下启用。

如果可能，它将以确定性方式混淆导出的名字。

迁移：不需要做什么。

## 命名 Chunk IDs

在开发模式下默认启用的一个新的 chunk id 的命名算法为 chunk id（和文件名）提供了更有可读性的名称。
模块 ID 根据其上下文的相对路径确定。
chunk ID 根据 chunk 的内容确定。

因此，您不再需要使用 `import(/* webpackChunkName: "name" */ "module")` 进行调试。
但是，如果要控制生产环境的文件名，这仍然有意义。

可以在生产中使用 `chunkIds: "named"`，但要确保不要意外泄露有关模块名称的敏感信息。

迁移：如果您不喜欢在开发中更改文件名，则可以传递 `chunkIds: "natural"` 以使用旧的数字模式。

## JSON 模块

JSON 模块现在符合规范，并在使用非默认导出时发出警告。

迁移：使用默认导出。

(从 alpha.16 开始)

未使用的属性通过 `optimization.usedExports` 优化被丢弃，而使用的属性通过 `optimization.mangleExports` 优化被混淆。（从 beta.3 开始）

当导入字符串 ES Module 时，JSON 模块不再具有导出名称（从 beta.7 开始）

可以在 `Rule.parser.parse` 中指定自定义 JSON 解析器，以导入类 JSON 文件（例如，toml，yaml，json5等）。（从 beta.8 开始）

##  嵌套地 tree-shaking

webpack 现在能够对导出的嵌套属性的访问进行跟踪，重新导出命名空间的对象时，可以改善 Tree Shaking（清除未使用的，且混淆使用的）。

``` js
// inner.js
export const a = 1;
export const b = 2;

// module.js
import * as inner from "./inner";
export { inner }

// user.js
import * as module from "./module";
console.log(module.inner.a);
```

在这个例子中，导出的 `b` 在生产环境中会被删除。

(从 alpha.15 开始)

##  内部模块 tree-shaking

webpack4 没有分析模块导出和导入之间的依赖关系。webpack5 有一个新的选项 `optimization.innerGraph`，该选项在生产模式下默认启用，它对模块中的字符进行分析以找出从导出到导入的依赖关系。

在这样的模块中：

``` js
import { something } from "./something";

function usingSomething() {
  return something;
}

export function test() {
  return usingSomething();
}
```

内部图算法将推断出仅在导出使用 `test` 时使用 `something`。这允许将更多导出标记为未使用，并在打包后省略更多代码。

如果设置了 `"sideEffects": false`，则可以省略更多模块。在这个例子中，当未使用导出的 `test` 时，将省略 `./something` 代码。

通过使用 `optimization.unusedExports` 可获得未使用的导出的信息，使用 `optimization.sideEffects` 可删除无副作用的模块。

以下字符会被分析：
* 函数声明
* 类声明
* `export default` 或变量声明以下内容
  * 函数表达式
  * 类表达式
  * `/*#__PURE__*/` 表达式
  * 局部变量
  * 导入绑定

反馈：如果您发现此分析中缺少某些内容，请通过一个 issue 反映，我们会考虑添加它。

此优化也称为深度范围分析。

(从 alpha.24 开始)

使用 `eval()` 可以为模块进行这种优化，因为 `eval` 后的代码可以引用作用域中的任何字符。

(从 beta.10 开始)

## CommonJs Tree Shaking

webpack 用于退出使用的导出分析，以分析 CommonJs 的导出和 `require()` 调用。

webpack5 增加了对某些 CommonJs 构造的支持，允许消除未使用的 CommonJs 导出，并从 `require()` 调用中跟踪引用的导出名称。

支持以下构造：

- `exports|this|module.exports.xxx = ...`
- `Object.defineProperty(exports|this|module.exports, "xxx", ...)`
- `require("abc").xxx`
- `require("abc").xxx()`
- importing from ESM
- `require()` a ESM
- 标记为 exportType（非严格ESM导入的特殊处理）：
  - `Object.defineProperty(exports|this|module.exports, "__esModule", { value: true|!0 })`
  - `exports|this|module.exports.__esModule = true|!0`

(从 beta.9 开始)

## 编译器空闲和关闭

现在需要在使用编译器后将其关闭。编译器在进入和离开空闲状态之后会带有这些状态的钩子。插件可以使用这些勾子执行不重要的工作。（比如，将持久性缓存缓慢地存储到磁盘）。编译器关闭时，所有剩余的工作应尽快完成，有一个回调会表明关闭已完成。

插件及其各自的作者应该期望某些用户可能会忘记关闭编译器。因此，所有工作最终也应该在空闲时完成。当工作完成时，应防止流程退出。

当传递回调时，`webpack()` 会自动调用 `close`.

迁移：在使用 node.js API 时，请确保在完成后调用 `Compiler.close`.

## 改进代码生成

现在有一个新的选项 `output.ecmaVersion`，它允许为 Webpack 生成的运行时代码指定 EcmaScript 版本。

webpack4 仅用于生成 ES5 代码。

webpack5 现在可以生成 ES5 和 ES6/ES2015 代码。

默认配置将生成 ES2015 代码。如果您需要支持较旧的浏览器（例如IE11），则可以将其降低为 `output.ecmaVersion：5`。

选择 `output.ecmaVersion: 2015` 将使用箭头函数生成较短的代码，并使用带有 TDZ 的 `const` 声明（用于导出默认值）生成更多符合规范的代码。

(从 alpha.23 开始)

生产模式中的默认最小化还使用此 `ecmaVersion` 选项生成更小的代码。（从 alpha.31 开始）

## SplitChunks 和模块大小

与显示单个数字相比，模块现在以更好的方式表示尺寸。此外，现在有不同类型的尺寸。

SplitChunksPlugin 现在知道如何处理这些不同的大小，并将它们用于 `minSize` 和 `maxSize`。
默认情况下，仅处理 `javascript` 的大小，但是您现在可以传递多个值来管理它们：

```js
minSize: {
  javascript: 30000,
  style: 50000,
}
```

迁移：检查构建中使用了哪种类型的大小，并在 `splitChunks.minSize` 和（可选）`splitChunks.maxSize` 中进行配置。

## 持久缓存

现在有一个文件系统缓存，它是可选的，可以通过以下配置启用：

``` js
cache: {
  // 1. Set cache type to filesystem
  type: "filesystem",
  
  buildDependencies: {
    // 2. Add your config as buildDependency to get cache invalidation on config change
    config: [__filename]
  
    // 3. If you have other things the build depends on you can add them here
    // Note that webpack, loaders and all modules referenced from your config are automatically added
  }
}
```

重要提示：

默认情况下，webpack 假定 webpack 所在的 `node_modules` 目录仅由包管理器修改。对于 `node_modules`，将跳过哈希和时间戳。出于性能原因，仅使用软件包名称和版本。当然使用符号链接（即 `npm/yarn 链接`）也没问题。除非您通过 `cache.managedPaths: []` 退出此优化，否则不要直接在 `node_modules` 中编辑文件。

使用 node_modules 时，缓存将默认存储在 `node_modules/.cache/webpack`中，使用 Yarn PnP 时，缓存将存储在 `.pnp / .cache / webpack` 中（从 alpha.21 开始）。您可能永远不必手动删除它。

（从 alpha.20 开始）

使用 Yarn PnP Webpack 时，假定 yarn 的缓存是不可变的（通常是不变的）。您可以通过 `cache.immutablePaths: []` 退出此优化。

（从 alpha.21 开始）

`SourceMapDevToolPlugin` 使用了持久性缓存。

（从 beta.4 开始）

`ConcatenatedModule` 使用了持久性缓存。

（从 beta.10 开始）

`ProgressPlugin` 使用了持久性缓存。

（从 beta.14 开始）

## 文件输出

webpack 过去总是在首次构建期间发出所有输出文件，但是在增量（监视）构建期间跳过写入未更改的文件。假定在运行 webpack 时没有其他更改输出的文件。

通过添加永久缓存，即使重新启动 webpack 进程，也应提供类似手表的体验，但是如果认为即使 webpack 不在运行，也没有其他更改输出目录，这将是一个过分的假设。

因此，webpack 现在将检查输出目录中的现有文件，并将其内容与内存中的输出文件进行比较，仅在文件更改后才写入文件。
这只会在首次构建时完成。在运行中的 webpack 进程中，当生成新资源时，任何增量构建都会始终将其写入文件。

我们假设 webpack 和插件仅在内容更改后才生成新资源，缓存用于确保输入相等时不会生成新资源。
不遵循此建议将降低性能。

当已经存在同名文件时，将永远不会写入标记为 “不可变”（包括内容哈希）的文件。
我们假设文件内容更改时，内容哈希将更改。一般而言，这是正确的，但在 webpack 或插件开发期间可能并不总是正确。

（从 beta.3 开始）

## 用于单个文件目标的 SplitChunks

现在，仅允许启动单个文件的目标（如 node, WebWorker, electron main）支持在运行时自动加载引导程序所需的从属文件。

这允许对带有 `chunks: "all"` 的目标使用 `splitChunks`.

请注意，由于块加载是异步的，因此这也会使初始计算也是异步的。 当使用 `output.library` 时这可能是一个问题，因为导出的值现在是一个 Promise。从alpha.14 开始，它不适用于 `target: "node"`，因为块加载在此处是同步的。

(从 alpha.3 开始)

## 更新后的解析器

`enhanced-resolve` 已更新至 v5，它具有以下改进：

- 使用 Yarn PnP 时，解析器无需其他插件直接处理它
- 解决方案跟踪更多依赖项，例如丢失文件
- 别名可能有多种选择
- 现在可以将别名设为 false
- 提高性能

(从 alpha.18 开始)

## 不包含 JS 的 Chunks

不包含 JS 代码的块将不再生成 JS 文件。

(从 alpha.14 开始)

## 实验性

并非所有功能从一开始就稳定。在 webpack4 中，我们添加了一些实验性功能，并在 changelog 中指出它们是实验性的，但是从配置中不一定能搞清楚哪些功能是实验性的。

在 webpack5 中，有一个新的 `experiments` 配置选项，可以启用实验性功能。这样可以明确启用或者使用了哪些试实验性功能。

尽管 webpack 遵循语义版本控制，但是实验性功能是个例外。实验性功能可能包含次要 webpack 版本中的重大更改。发生这种情况时，我们将在更改日志中添加清晰的注释。这将使我们能够更快地完成实验性功能，同时也使我们能够在主要版本上停留更长时间以获得稳定的功能。

webpack5 将会带有以下实验性功能：

* 和 webpack4 一样支持 `.mjs`（`experiments.mjs`）
* 和 webpack4 一样支持旧的WebAssembly（`experiments.syncWebAssembly`）
* 根据 [更新后的规范]（https://github.com/WebAssembly/esm-integration）支持新的 WebAssembly（`experiments.asyncWebAssembly`）
    * 这使 WebAssembly 模块成为异步模块
* [Top Level Await]（https://github.com/tc39/proposal-top-level-await）Stage3 提案（`experiments.topLevelAwait`）
    * 在顶层使用 `await`，将使模块变为异步模块
* 使用 `import` 导入异步模块（`experiments.importAsync`）
* 使用`import await` 导入异步模块（`experiments.importAwait`）
* `asset` 模块类型，类似于 `file-loader` | `url-loader` | `raw-loader`（`experiments.asset`）（从 alpha.19 开始）
    * 从 beta.8 开始支持资源路径和与此相关的选项
* 将包作为模块输出（`experiments.outputModule`）（从 alpha.31 开始）
    * 从包中删除了包装后的 IIFE，强制执行严格模式，通过 `<script type="module">` 进行延迟加载，并在 module 类型下将其最小化

注意，这也意味着默认情况下禁用 `.mjs` 和 WebAssembly 的支持。

(从 alpha.15 开始)

## 统计资料

默认情况下，chunk 的关系被隐藏，可以通过 `stats.chunkRelations` 来切换。

(从 alpha.1 开始)

统计信息现在可以区分 `files` 和 `auxiliaryFiles`.

(从 alpha.19 开始)

统计信息默认情况下现在隐藏模块和 chunk ids，可以使用 `stats.ids` 来切换。

现在，所有模块的列表均按到入口点的距离排序，这可以通过 `stats.modulesSort` 来更改。

各个 chunk 的根模块下的 chunk 模块列表按模块名称排序，这可以通过各自的 `stats.chunkModulesSort` 和 `stats.chunkRootModulesSort` 来更改。

现在，串联模块中的嵌套模块列表是按照拓扑排序，这可以通过 `stats.nestedModulesSort` 来更改。

Chunks 和资源现在会展示 chunk id 提示。

(从 alpha.31 开始)

## 进展

CLI 对使用 `--progress` 的 `ProgressPlugin` 进行了一些改进，但也可以手动用作插件。

它仅用于统计已处理的模块，现在，它可以统计 `entries` `dependencies` 和 `modules`.
现在这些都会被默认显示。

它用于禁用当前处理的模块，这导致大量进程 stderr 输出，并在某些控制台上产生了性能问题。

现在默认是禁用的（`activeModules` 选项）。这也减少了控制台上的垃圾邮件数量。（从 alpha.31 开始）

现在对 stderr 的写入被限制为 500 ms。

(从 beta.4 开始)

分析模式也进行了升级，将显示嵌套进度消息的时间。
这使得在插件引起性能问题时更容易找出原因。

(从 beta.10 开始)

添加了 `percentBy` 选项，该选项告诉 `ProgressPlugin` 如何计算进度百分比。

```js
new webpack.ProgressPlugin({ percentBy: "entries" });
```

为了使进度百分比更准确，`ProgressPlugin` 会缓存最近已知的模块总数，并在下一个构建中再次使用该值。第一个版本将预热缓存，但随后的版本将使用并更新该值。

(从 beta.14 开始)

## 最低的 Node.js 版本

最低支持的 node.js 版本已从 6 增加到 10.13.0（LTS）。

迁移：升级到最新的 node.js 版本。