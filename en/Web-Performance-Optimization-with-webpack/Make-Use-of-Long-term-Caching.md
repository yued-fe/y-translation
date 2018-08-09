# Make use of long-term caching

# 利用好持久化缓存

> - **原文地址：** https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Make-Use-of-Long-term-Caching.md
> - **译者：** 周文康
> - **校对者：**

The next thing (after [optimizing the app size](./decrease-frontend-size)) that improves the app loading time is caching. Use it to keep parts of the app on the client and avoid re-downloading them every time.

在[优化应用体积](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size)之后,下一个提升应用加载时间的策略就是缓存。在客户端中使用缓存作为应用的一部分，可以避免之后每次的重复下载。

## Use bundle versioning and cache headers

## bundle 的版本控制和缓存头的使用

The common approach of doing caching is to:

使用缓存的通用方法：

 1. tell the browser to cache a file for a very long time (e.g., a year):

 1. 告诉浏览器需要缓存一个文件很长时间（比如，一年）

    ``` js
    # Server header
    Cache-Control: max-age=31536000
    ```

    Note: If you aren’t familiar what `Cache-Control` does, see Jake Archibald’s excellent post [on caching best
    practices](https://jakearchibald.com/2016/caching-best-practices/).
    
    注意：如果你不熟悉 Cache-Control 的原理，请参阅 Jake Archibald 的文章: [关于缓存的最佳实践](https://jakearchibald.com/2016/caching-best-practices/)
    
2. and rename the file when it’s changed to force the re-download:

2. 可以通过修改文件名强制去执行重新下载

    ``` js
    !-- Before the change -->
    <script src="./index-v15.js"></script>
    
    !-- After the change -->
    <script src="./index-v16.js"></script>
    
    ```

This approach tells the browser to download the JS file, cache it and use the cached copy. The browser will only hit the network only if the file name changes (or if a year passes).

这个方法可以告诉浏览器去下载 JS 文件，并将它缓存，之后使用的都是它的缓存副本。浏览器只会在文件名发生改变时才会请求网络（或者一年之后缓存失效）。

With webpack, you do the same, but instead of a version number, you specify the file hash. To include the hash into the file name, use
[`[chunkhash]`](https://webpack.js.org/configuration/output/#output-filename):

使用webpack，你可以做同样的事，但不是使用版本号，而是指定文件的哈希值。使用 [`[chunkhash]`](https://webpack.js.org/configuration/output/#output-filename) 可以将哈希值写入文件名中：
``` js
// webpack.config.js
module.exports = {
  entry: './index.js',
  output: {
    filename: 'bundle.<strong>[chunkhash]</strong>.js',
        // → bundle.8e0d62a03.js
  },
};
```

> ⭐️ Note: Webpack could generate a different hash even if the bundle stays the same – e.g. if you rename a file or compile the bundle under a different OS. This is a bug, and there’s no clear solution yet. [See the discussion on GitHub](https://github.com/webpack/webpack/issues/1479)

> ⭐️ 注意: 即使 bundle 不变，webpack 也可能生成不同的哈希值 – 例如，你重命名了一个文件或者在不同的操作系统下编译了 bundle。 这是一个 bug.
还没有明确的解决方案。[具体可参阅GitHub上的讨论](https://github.com/webpack/webpack/issues/1479)

If you need the file name to send it to the client, use either the `HtmlWebpackPlugin` or the `WebpackManifestPlugin`.

如果你需要将文件名发送给客户端，可以使用 `HtmlWebpackPlugin` 或者 `WebpackManifestPlugin`。

The [`HtmlWebpackPlugin`](https://github.com/jantimon/html-webpack-plugin) is a simple, but less flexible approach. During compilation, this plugin generates an HTML file which includes all compiled resources. If your server logic isn’t complex, then it should be enough for you:

[`HtmlWebpackPlugin`](https://github.com/jantimon/html-webpack-plugin)是一个简单但扩展性不强的插件。在编译期间，它会生成一个包含所有编译已资源的HTML文件。如果你的服务端逻辑不是很复杂，那么它应该就能满足你：

```js
<!-- index.html -->
<!doctype html>
<!-- ... -->
<script src="bundle.8e0d62a03.js"></script>
```

The [`WebpackManifestPlugin`](https://github.com/danethurber/webpack-manifest-plugin) is a more flexible approach which is useful if you have a complex server part. During the build, it generates a JSON file with a mapping between file names without hash and file names with hash. Use this JSON on the server to find out which file to work with:

[`WebpackManifestPlugin`](https://github.com/danethurber/webpack-manifest-plugin)是一个扩展性更佳的插件，它可以帮助你解决服务端逻辑比较复杂的那部分。在编译阶段，它会生成一个JSON文件，里面包含了原文件名和带哈希文件名的映射。在服务端，通过这个JSON就能方便的找到我们真正要执行的文件：
``` js
// manifest.json
{
  "bundle.js": "bundle.8e0d62a03.js"
}
```

### Further reading

### 扩展阅读

* Jake Archibald [about caching best practices](https://jakearchibald.com/2016/caching-best-practices/)

* Jake Archibald [关于缓存的最佳实践](https://jakearchibald.com/2016/caching-best-practices/)

## Extract dependencies and runtime into a separate file

## 将依赖项和运行时提取到单独的文件中

### Dependencies

### 依赖项

App dependencies tend to change less often than the actual app code. If you move them into a separate file, the browser will be able to cache them separately – and won’t re-download them each time the app code changes.

应用的依赖项通常比实际应用内的代码变更频率低。如果你将它们移到一个独立的文件中，浏览器可以把它们独立缓存起来 – 同时每次应用代码变更也不会去重新下载。

> Key Term: In webpack terminology, separate files with the app code are called *chunks*. We’ll use this name later.

> 关键术语：在 webpack 术语中，把带有应用程序代码的独立文件称之为 *chunk*。我们稍后会使用这个名称。

To extract dependencies into a separate chunk, perform three steps:

要将依赖项提取到独立的块中，需要执行下面三个步骤：

1. Replace the output filename with `[name].[chunkname].js`:

1. 将输出文件名替换为`[name].[chunkname].js`：

    ``` js
    // webpack.config.js
    module.exports = {
      output: {
        // Before
        filename: 'bundle.[chunkhash].js',
        // After
        filename: '[name].[chunkhash].js',
      },
    };
    ```
    
    When webpack builds the app, it replaces [`[name]`](https://webpack.js.org/configuration/output/#output-filename) with a name of a chunk. If we don’t add the `[name]` part, we’ll have to differentiate between chunks by their hash – which is pretty hard!

  当 webpack 编译应用时，它会将[`[name]`](https://webpack.js.org/configuration/output/#output-filename) 作为 chunk 的名称。如果我们没有添加 `[name]`的部分，我们将不得不通过哈希值来区分 chunk - 这样就变得非常困难！
  
2. Convert the `entry` field into an object:

2. 将`entry`的值改为对象：
    ``` js
    // webpack.config.js
    module.exports = {
      // Before
      entry: './index.js',
      // After
      entry: {
        main: './index.js',
      },
    };
    ```

    In this snippet, “main” is a name of a chunk. This name will be substituted in place of `[name]` from step 1.
    
    在上面这段代码中，“main” 是 chunk 的名称。这个名称会在第一步时被`[name]`所替代。
    
    By now, if you build the app, this chunk will include the whole app code – just like we haven’t done these steps. But this will change in a sec.
    
    到目前为止，如果你编译应用，这个 chunk 还是包含了整个应用的代码 - 就像我们没有做过上述这些步骤一样。但接下来很快就将产生变化。
    
3. **In webpack 4,** add the `optimization.splitChunks.chunks: 'all'` option into your webpack config:

3. **在 webpack 4 中,** ，可以将`optimization.splitChunks.chunks: 'all'` 选项添加到 webpack 的配置中:

    ``` js
    // webpack.config.js (for webpack 4)
    module.exports = {
      optimization: {
        splitChunks: {
          chunks: 'all',
        }
      },
    };
    ```

    This option enables smart code splitting. With it, webpack would extract the vendor code if it gets larger than 30 kB (before minification and gzip). It would also extract the common code – this is useful if your build produces several bundles (e.g. [if you split your app into routes](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching#split-the-code-into-routes-and-pages))
    
    这个选项可以开启智能代码拆分。使用了这个功能，webpack 将会把大于 30KB（压缩和gzip之后） 的代码提取到 vendor (公共库)代码中。它同时也可以提取 common 代码 - 如果你需要编译生成多个 bundles 时这将非常有用。（例如：[假如你通过路由来拆分应用](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching#split-the-code-into-routes-and-pages)）。
    
    **In webpack 3 ** add the [CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/):
     
    **在 webpack 3 中** 可以添加 [CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/)插件:
    ``` js
    // webpack.config.js (for webpack 3)
    module.exports = {
      plugins: [
        new webpack.optimize.CommonsChunkPlugin({
          // A name of the chunk that will include the dependencies.
          // This name is substituted in place of [name] from step 1
          name: 'vendor',
    
          // A function that determines which modules to include into this chunk
          minChunks: module => module.context &&
            module.context.includes('node_modules'),
        }),
      ],
    };
    ```
    
    This plugin takes all modules which paths include `node_modules` and moves them into a separate file called `vendor.[chunkhash].js`.
    这个插件会将`node_modules`下的所有模块移到一个名为 vendor.[chunkhash].js 的独立文件中。


After these changes, each build will generate two files instead of one: `main.[chunkhash].js` and `vendor.[chunkhash].js` (`vendors~main.[chunkhash].js` for webpack 4). In case of webpack 4, the vendor bundle might not be generated if dependencies are small – and that’s fine:

完成这些更改后，每次编译将不止生成一个文件，而生成了两个文件：`main.[chunkhash].js` 和`vendor.[chunkhash].js` (`vendors~main.[chunkhash].js` 只有在 webpack 4 才有)。在 webpack 4 中，如果依赖项很少，则可能不会生成 vendor bundle - 这点做的不错：

``` js
$ webpack
Hash: ac01483e8fec1fa70676
Version: webpack 3.8.1
Time: 3816ms
                           Asset   Size  Chunks             Chunk Names
  ./main.00bab6fd3100008a42b0.js  82 kB       0  [emitted]  main
./vendor.d9e134771799ecdf9483.js  47 kB       1  [emitted]  vendor
```

The browser would cache these files separately – and redownload only code that changes.

浏览器会单独缓存这些文件 - 同时只有代码发生改变时才会重新下载。

### Webpack runtime code

### Webpack 运行时代码

Unfortunately, extracting just the vendor code is not enough. If you try to change something in the app code:

当然，仅仅提取 vendor 代码还是不够的。如果你项尝试在应用代码中修改一些东西：
``` js
// index.js
…
…

// E.g. add this:
console.log('Wat');
```

you’ll notice that the `vendor` hash also changes:

你会发现`vendor`的哈希值也会被改变：

``` js
                           Asset   Size  Chunks             Chunk Names
./vendor.d9e134771799ecdf9483.js  47 kB       1  [emitted]  vendor
```

↓

``` js
                            Asset   Size  Chunks             Chunk Names
./vendor.e6ea4504d61a1cc1c60b.js  47 kB       1  [emitted]  vendor
```

This happens because the webpack bundle, apart from the code of modules, has *[a runtime](https://webpack.js.org/concepts/manifest/)* – a small piece of code that manages the module execution. When you split the code into multiple files, this piece of code starts including a mapping between chunk ids and corresponding files:

这是由于 webpack 打包时，一部分模块的代码拥有*[a runtime](https://webpack.js.org/concepts/manifest/)*  - 一小段可以管理模块执行的代码。当你将代码拆分成多个文件时，这小部分代码在 chunk ids 和 匹配的文件之间会生成一个映射：

``` js
// vendor.e6ea4504d61a1cc1c60b.js
script.src = __webpack_require__.p + chunkId + "." + {
  "0": "2f2269c7f0a55a5c1871"
}[chunkId] + ".js";
```
Webpack includes this runtime into the last generated chunk, which is `vendor` in our case. And every time any chunk changes, this piece of code changes too, causing the whole `vendor` chunk to change.

Webpack 将运行时包含在了最新生成的 chunk 中，这个 chunk 就是我们代码中的 `vendor`。每次 chunk 有任何变更，这一小部分代码也会随之更改，同时也会导致整个 `vendor` chunk 发生改变。

To solve this, let’s move the runtime into a separate file. **In webpack 4,** this is achieved by enabling the `optimization.runtimeChunk` option:

为了解决这个问题，我们可以将运行时移动到一个独立的文件中。**在 webpack 4 中**，可以通过开启 `optimization.runtimeChunk` 选项来实现：

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    runtimeChunk: true,
  },
};
```

**In webpack 3** do this by creating an extra empty chunk with the `CommonsChunkPlugin`:

**在 webpack 3 中**，可以通过 `CommonsChunkPlugin` 创建一个额外的空 chunk：

``` js
// webpack.config.js (for webpack 3)
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',

      minChunks: module => module.context &&
        module.context.includes('node_modules'),
    }),

    // This plugin must come after the vendor one (because webpack
    // includes runtime into the last chunk)
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime',

      // minChunks: Infinity means that no app modules
      // will be included into this chunk
      minChunks: Infinity,
    }),
  ],
};
```
After these changes, each build will be generating three files:

完成这些变更后，每次编译将生成三个文件：

``` js
$ webpack
Hash: ac01483e8fec1fa70676
Version: webpack 3.8.1
Time: 3816ms
                            Asset     Size  Chunks             Chunk Names
   ./main.00bab6fd3100008a42b0.js    82 kB       0  [emitted]  main
 ./vendor.26886caf15818fa82dfa.js    46 kB       1  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
```

Include them into `index.html` in the reverse order – and you’re done:

将这个文件按倒序的方式添加到 `index.html` 中，就完成了：

``` js
<!-- index.html -->
<script src="./runtime.79f17c27b335abc7aaf4.js"></script>
<script src="./vendor.26886caf15818fa82dfa.js"></script>
<script src="./main.00bab6fd3100008a42b0.js"></script>
```

### Further reading

### 扩展阅读

* Webpack guide [on long term caching](https://webpack.js.org/guides/caching/)

* Webpack 指南 [关于持久化缓存](https://webpack.js.org/guides/caching/)

* Webpack docs [about webpack runtime and
  manifest](https://webpack.js.org/concepts/manifest/)
  
* Webpack 文档 [关于webpack的运行时和manifest](https://webpack.js.org/concepts/manifest/)
  
* [“Getting the most out of the
  CommonsChunkPlugin”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)

* [“优化高效的使用 CommonsChunkPlugin”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)
  
* [How `optimization.splitChunks` and `optimization.runtimeChunk` work](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

* [`optimization.splitChunks` 和 `optimization.runtimeChunk` 的工作原理](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

## Inline webpack runtime to save an extra HTTP request

## 内联 webpack 的运行时可以节省额外的 HTTP 请求

To make things even better, try inlining the webpack runtime into the HTML response. I.e., instead of this:

为了达到更好的体验，我们可以尝试把 webpack 的运行时内联到 HTML 中。例如，我们不要这么做：

``` js
<!-- index.html -->
<script src="./runtime.79f17c27b335abc7aaf4.js"></script>
```

而是像下面这样:

``` js
<!-- index.html -->
<script>
!function(e){function n(r){if(t[r])return t[r].exports;…}} ([]);
</script>
```

The runtime is small, and inlining it will help you save an HTTP request (pretty important with HTTP/1; less important with HTTP/2 but might still play an effect).

运行时的代码不多，内联到 Html 中可以帮助我们节省 HTTP 请求（在 HTTP/1 中尤为重要；在 HTTP/2 中虽然没那么重要，但仍然能起到一定作用）

Here’s how to do it.

下面就来看看要如何做呢。

### If you generate HTML with the HtmlWebpackPlugin

### 如果你使用 HtmlWebpackPlugin 插件来生成 HTML

If you use the [HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin) to generate an HTML file, the [InlineSourcePlugin](https://github.com/DustinJackson/html-webpack-inline-source-plugin) is all you need:

如果你使用 [HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin) 插件来生成 HTML 文件，那么 [InlineSourcePlugin](https://github.com/DustinJackson/html-webpack-inline-source-plugin) 插件就能帮你完成：
``` js
// webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const InlineSourcePlugin = require('html-webpack-inline-source-plugin');

module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      // Inline all files which names start with “runtime~” and end with “.js”.
      // That’s the default naming of runtime chunks
      inlineSource: 'runtime~.+\\.js',
    }),
    // This plugin enables the “inlineSource” option
    new InlineSourcePlugin(),
  ],
};
```
### If you generate HTML using a custom server logic

### 如果你使用的是自定义的服务端逻辑来生成 HTML

**With webpack 4:**

**在 webpack 4 中:**

1. Add the [WebpackManifestPlugin](https://github.com/danethurber/webpack-manifest-plugin) to know the generated name of the runtume chunk:

1. 添加[WebpackManifestPlugin](https://github.com/danethurber/webpack-manifest-plugin)插件可以获取运行时 chunk 的生成名称：
    ``` js
    // webpack.config.js (for webpack 4)
    const ManifestPlugin = require('webpack-manifest-plugin');
    
    module.exports = {
      plugins: [
        new ManifestPlugin(),
      ],
    };
    ```

    A build with this plugin would create a file that looks like this:
    
    使用这个插件编译会生成像下面这样的文件：
    
    ``` js
    // manifest.json
    {
      "runtime~main.js": "runtime~main.8e0d62a03.js"
    }
    ```

2. Inline the content of the runtime chunk in a convenient way. E.g. with Node.js and Express:

2. 用一个方便的方式内联运行时 chunk 的内容。例如，可以使用 Node.js 和 Express：
    ``` js
    // server.js
    const fs = require('fs');
    const manifest = require('./manifest.json');
    
    const runtimeContent = fs.readFileSync(manifest['runtime~main.js'], 'utf-8');
    
    app.get('/', (req, res) => {
      res.send(`
        …
        &lt;script>${runtimeContent}&lt;/script>
        …
      `);
    });
    ```

**Or with webpack 3:**

**在 webpack 3 中:**

1. Make the runtime name static by specifying `filename`:

1. 指定运行时的名称为`filename`:

    ``` js
    // webpack.config.js (for webpack 3)
    module.exports = {
      plugins: [
        new webpack.optimize.CommonsChunkPlugin({
          name: 'runtime',
          minChunks: Infinity,
          filename: 'runtime.js',
            // → Now the runtime file will be called
            // “runtime.js”, not “runtime.79f17c27b335abc7aaf4.js”
        }),
      ],
    };
    ```

2. Inline the <code>runtime.js</code> content in a convenient way. E.g. with Node.js and Express:

2. 用一个方便的方式内联<code>runtime.js</code>的内容。例如，可以使用 Node.js 和 Express：

    ``` js
    // server.js
    const fs = require('fs');
    const runtimeContent = fs.readFileSync('./runtime.js', 'utf-8');
    
    app.get('/', (req, res) => {
      res.send(`
        …
        &lt;script>${runtimeContent}&lt;/script>
        …
      `);
    });
    ```

## Lazy-load code that you don’t need right now

## 懒加载

Sometimes, a page has more and less important parts:

通常，一个网页会有自身的侧重点：

* If you load a video page on YouTube, you care more about the video than about
  comments. Here, the video is more important than comments.
  
* 假如你在 YouTube 上加载一个视频页面, 你更关心的肯定是视频而不是评论. 所以, 这里视频就比评论重要。
  
* If you open an article on a news site, you care more about the text of the
  article than about ads. Here, the text is more important than ads.

* 又比如你在一个新闻网站看一篇文章, 你更关心的肯定是文章的文字而不是广告. 所以, 这里文字就比广告重要。

In such cases, improve the initial loading performance by downloading only the most important stuff first, and lazy-loading the remaining parts later. Use [the `import()` function](https://webpack.js.org/api/module-methods/#import-) and
[code-splitting](https://webpack.js.org/guides/code-splitting/) for this:

上述的这些案例，都是通过优先下载最重要的部分，稍后懒加载剩余部分，从而来提升页面首次加载的性能。在webpack中，使用[`import()` 函数](https://webpack.js.org/api/module-methods/#import-) 和 [code-splitting](https://webpack.js.org/guides/code-splitting/)即可实现。

``` js
// videoPlayer.js
export function renderVideoPlayer() { … }

// comments.js
export function renderComments() { … }

// index.js
import {renderVideoPlayer} from './videoPlayer';
renderVideoPlayer();

// …Custom event listener
onShowCommentsClick(() => {
  import('./comments').then((comments) => {
    comments.renderComments();
  });
});
```
`import()` specifies that you want to load a specific module dynamically. When webpack sees `import('./module.js')`, it moves this module into a separate chunk:

`import()`函数可以帮助你实现按需加载。webpack 在打包时遇到 `import('./module.js')`，就会把这个模块放到单独的 chunk 中：


``` js
$ webpack
Hash: 39b2a53cb4e73f0dc5b2
Version: webpack 3.8.1
Time: 4273ms
                            Asset     Size  Chunks             Chunk Names
      ./0.8ecaf182f5c85b7a8199.js  22.5 kB       0  [emitted]
   ./main.f7e53d8e13e9a2745d6d.js    60 kB       1  [emitted]  main
 ./vendor.4f14b6326a80f4752a98.js    46 kB       2  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
```

and downloads it only when execution reaches the `import()` function.

只有当代码执行到 `import()` 函数时才会去下载。

This will make the `main` bundle smaller, improving the initial loading time.
Even more, it will improve caching – if you change the code in the main chunk,
comments chunk won’t get affected.

这样可以让 `入口` bundle 变得更小，从而减少首次加载时间。不仅如此，它还可以优化缓存 - 如果你修改了入口 chunk 的代码，注释 chunk 不会受到影响。

> ⭐️ Note: If you compile this code with Babel, you’ll have a syntax error because Babel doesn’t understand `import()` out of the box. To avoid the error, add the [`syntax-dynamic-import`](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import) plugin.

> ⭐️ 注意: 如果你使用 Babel 编译代码，会因为 Babel 无法识别 `import()` 而出现语法错误。为了避免这个错误，你可以添加[`syntax-dynamic-import`](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import)插件。

### Further reading

### 扩展阅读

* Webpack docs [for the `import()` function](https://webpack.js.org/api/module-methods/#import-)

* Webpack 文档 [`import()` 函数的使用](https://webpack.js.org/api/module-methods/#import-)

* The JavaScript proposal [for implementing the `import()` syntax](https://github.com/tc39/proposal-dynamic-import)

* JavaScript 提案  [实现`import()`语法](https://github.com/tc39/proposal-dynamic-import)

## Split the code into routes and pages

## 将代码拆分为路由和页面

If your app has multiple routes or pages, but there’s only a single JS file with the code (a single `main` chunk), it’s likely that you’re serving extra bytes on each request. For example, when a user visits a home page of your site:

如果你的应用有多个路由或页面，但是代码中只有单独一个的 JS 文件（一个单独的`入口` chunk），这样似乎会让你的每次请求都附加了额外的流量。例如，当用户访问你网站的首页：

![](https://developers.google.com/web/fundamentals/performance/webpack/site-home-page.png)

they don’t need to load the code for rendering an article that’s on a different page – but they will load it. Moreover, if the user always visits only the home page, and you make a change in the article code, webpack will invalidate the whole bundle – and the user will have to re-download the whole app.

他们并不需要加载其它页面上用于渲染文章的代码 - 但他们却加载了。此外，如果这个用户经常只是访问首页，但如果你更改了其它页面的文章代码，webpack 会重新编译，使整个 bundle 失效 - 这样导致用户将重新下载整个应用的代码。

If we split the app into pages (or routes, if it’s a single-page app), the user will download only the relevant code. Plus, the browser will cache the app code better: if you change the home page code, webpack will invalidate only the corresponding chunk.

如果我们将代码拆分到页面中（或者单页面应用的路由里），用户就会只下载真正用到的那部分代码。此外，浏览器也会更好地缓存应用代码：当你改变首页的代码时，webpack 只会让相匹配的 chunk 失效。

### For single-page apps

### 单页面应用

To split single-page apps by routes, use `import()` (see the [“Lazy-load code that you don’t need right now”](#lazy-loading) section). If you use a framework, it might have an existing solution for this:

要通过路由来拆分单页应用，可以使用 `import()`（参加上文[懒加载](#lazy-loading)部分）。如果你使用的是一个框架，目前也有现成的解决方案：

* [“Code Splitting”](https://reacttraining.com/react-router/web/guides/code-splitting) in `react-router`'s docs (for React)

* `react-router`文档中的[“Code Splitting”](https://reacttraining.com/react-router/web/guides/code-splitting)  (适用于React)

* `vue-router`文档中的[“Lazy Loading Routes”](https://router.vuejs.org/en/advanced/lazy-loading.html)(适用于Vue.js)

### For traditional multi-page apps

### 传统多页应用

To split traditional apps by pages, use webpack’s [entry points](https://webpack.js.org/concepts/entry-points/). If your app has three kinds of pages: the home page, the article page and the user account page, – it should have three entries:

要通过页面来拆分传统应用，可以使用 webpack 的 [entry points](https://webpack.js.org/concepts/entry-points/)。假设你的应用中有三类页面：主页、文章页、用户账户页，- 那么就应该有三个入口：
``` js
// webpack.config.js
module.exports = {
  entry: {
    home: './src/Home/index.js',
    article: './src/Article/index.js',
    profile: './src/Profile/index.js'
  },
};
```
For each entry file, webpack will build a separate dependency tree and generate a bundle that includes only modules that are used by that entry:

对于每个入口文件，webpack 将构建一个单独的依赖树并生成一个 bundle，这个 bundle 里只有包含这个入口所使用到的模块：

``` js
$ webpack
Hash: 318d7b8490a7382bf23b
Version: webpack 3.8.1
Time: 4273ms
                            Asset     Size  Chunks             Chunk Names
      ./0.8ecaf182f5c85b7a8199.js  22.5 kB       0  [emitted]
   ./home.91b9ed27366fe7e33d6a.js    18 kB       1  [emitted]  home
./article.87a128755b16ac3294fd.js    32 kB       2  [emitted]  article
./profile.de945dc02685f6166781.js    24 kB       3  [emitted]  profile
 ./vendor.4f14b6326a80f4752a98.js    46 kB       4  [emitted]  vendor
./runtime.318d7b8490a7382bf23b.js  1.45 kB       5  [emitted]  runtime
```

So, if only the article page uses Lodash, the `home` and the `profile` bundles won’t include it – and the user won’t have to download this library when visiting the home page.

所以，如果你的文章页面使用到了 Lodash， `home` and `profile` bundle其实不会包含它 - 用户也不会在访问首页的时候下载到这个库。

Separate dependency trees have their drawbacks though. If two entry points use Lodash, and you haven’t moved your dependencies into a vendor bundle, both entry points will include a copy of Lodash. To solve this, **in webpack 4,** add the `optimization.splitChunks.chunks: 'all'` option into your webpack config:

但是，单独的依赖树有它们的缺点。如果两个入口都使用到了 Lodash，同时你没有将依赖移到 vendor bundle中，则两个入口都将包含 Lodash 的副本。为了解决这个问题，**在 webpack 4 中**，可以在你的 webpack 配置中加入`optimization.splitChunks.chunks: 'all'`选项：
``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
    }
  },
};
```

This option enables smart code splitting. With this option, webpack would automatically look for common code and extract it into separate files.

这个选项可以开启智能代码拆分。有了这个选项，webpack 将自动查找到公共代码，并且提取待单独的文件中。

Or, **in webpack 3,** use the [`CommonsChunkPlugin`](https://webpack.js.org/plugins/commons-chunk-plugin/) – it will move common dependencies into a new specified file:

**在 webpack 3中**，可以使用[`CommonsChunkPlugin`](https://webpack.js.org/plugins/commons-chunk-plugin/)插件，它会将公共的依赖项移动到一个新的指定文件中：

``` js
// webpack.config.js (for webpack 3)
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      // A name of the chunk that will include the common dependencies
      name: 'common',

      // The plugin will move a module into a common file
      // only if it’s included into `minChunks` chunks
      // (Note that the plugin analyzes all chunks, not only entries)
      minChunks: 2,    // 2 is the default value
    }),
  ],
};
```

Feel free to play with the `minChunks` value to find the best one. Generally, you want to keep it small, but increase if the number of chunks grows. For example, for 3 chunks, `minChunks` might be 2, but for 30 chunks, it might be 8 – because if you keep it at 2, too many modules will get into the common file, inflating it too much.

你可以尝试调整`minChunks`的值来找到最优的效果。通常情况下，你希望它是一个较小的值，但随着 chunk 数量的增加它会随之增大。例如，有3个 chunk 时，`minChunks` 的值可能是 2 ，但是有30个 chunk 时，它的值可能是8 - 因为如果你把它设置成 2，就会有很多模块要被打包进同一个通用文件中，这样文件就会变得臃肿。

### Further reading 

### 扩展阅读

* Webpack docs [about the concept of entry points](https://webpack.js.org/concepts/entry-points/)

* Webpack 文档 [关于 entry points 的概念](https://webpack.js.org/concepts/entry-points/)

* Webpack docs [about the CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/)

* Webpack 文档 [关于 CommonsChunkPlugin 插件](https://webpack.js.org/plugins/commons-chunk-plugin/)

* [“Getting the most out of the CommonsChunkPlugin”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)

* [“CommonsChunkPlugin的最佳实践”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)

* [How `optimization.splitChunks` and `optimization.runtimeChunk` work](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

* [`optimization.splitChunks` 和 `optimization.runtimeChunk` 的工作原理](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

## Make module ids more stable

## 确保模块的 id 更加稳定

When building the code, webpack assigns each module an ID. Later, these IDs are used in `require()`s inside the bundle. You usually see IDs in the build output right before the module paths:

编译代码时，webpack 会为每个模块分配一个ID。随后，这些 ID 将在 bundle 里的 `require()` 函数中被使用到。你通常会在编译输出的模块路径前看到这些 ID：
``` bash
$ webpack
Hash: df3474e4f76528e3bbc9
Version: webpack 3.8.1
Time: 2150ms
                           Asset      Size  Chunks             Chunk Names
      ./0.8ecaf182f5c85b7a8199.js  22.5 kB       0  [emitted]
   ./main.4e50a16675574df6a9e9.js    60 kB       1  [emitted]  main
 ./vendor.26886caf15818fa82dfa.js    46 kB       2  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ 看下面

``` bash
   [0] ./index.js 29 kB {1} [built]
   [2] (webpack)/buildin/global.js 488 bytes {2} [built]
   [3] (webpack)/buildin/module.js 495 bytes {2} [built]
   [4] ./comments.js 58 kB {0} [built]
   [5] ./ads.js 74 kB {1} [built]
    + 1 hidden module
```

By default, IDs are calculated using a counter (i.e. the first module has ID 0, the second one has ID 1, and so on). The problem with this is that when you add a new module, it might appear in the middle of the module list, changing all the next modules’ IDs:

默认情况下，这些 ID 是使用计数器计算出来的（例如，第一个模块的 ID 是 0，第二个模块的 ID 就是 1，以此类推）。但这样做有个问题，当你新增一个模块时，它会可能出现在模块列表的中间，从而导致之后所有模块的 ID 都被改变：

``` js
$ webpack
Hash: df3474e4f76528e3bbc9
Version: webpack 3.8.1
Time: 2150ms
                           Asset      Size  Chunks             Chunk Names
      ./0.5c82c0f337fcb22672b5.js    22 kB       0  [emitted]
   ./main.0c8b617dfc40c2827ae3.js    82 kB       1  [emitted]  main
 ./vendor.26886caf15818fa82dfa.js    46 kB       2  [emitted]  vendor
./runtime.79f17c27b335abc7aaf4.js  1.45 kB       3  [emitted]  runtime
   [0] ./index.js 29 kB {1} [built]
   [2] (webpack)/buildin/global.js 488 bytes {2} [built]
   [3] (webpack)/buildin/module.js 495 bytes {2} [built]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ We’ve added a new module...

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ 我们添加了一个新模块...

``` js
   [4] ./webPlayer.js 24 kB {1} [built]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ And look what it has done! `comments.js` now has ID 5 instead of 4

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ 看看下面做了什么！ `comments.js` 的 ID 由 4 变成了 5

``` js
   [5] ./comments.js 58 kB {0} [built]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ `ads.js` now has ID 6 instead of 5

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ `ads.js` 的 ID 由 5 变成了 6

``` JS
   [6] ./ads.js 74 kB {1} [built]
       + 1 hidden module
``` 

This invalidates all chunks that include or depend on modules with changed IDs – even if their actual code hasn’t changed. In our case, the `0` chunk (the chunk with `comments.js`) and the `main` chunk (the chunk with the other app code) get invalidated – whereas only the `main` one should’ve been.

这将使包含或依赖于这些被更改 ID 的模块的所有 chunks 都无效 - 即使它们实际代码没有更改。在我们的案例中，`0` chunk ( `comments.js` 的 chunk)  和 `main` chunk （其它应用代码的 chunk ）都将失效 - 但其实只有 `main` 应该失效。

To solve this, change how module IDs are calculated using the [`HashedModuleIdsPlugin`](https://webpack.js.org/plugins/hashed-module-ids-plugin/). It replaces counter-based IDs with hashes of module paths:

为了解决这个问题，可以使用[`HashedModuleIdsPlugin`](https://webpack.js.org/plugins/hashed-module-ids-plugin/)插件来改变模块 ID 的计算方式。这个插件用模块路径的哈希值代替了基于计数器的ID：

``` js
$ webpack
Hash: df3474e4f76528e3bbc9
Version: webpack 3.8.1
Time: 2150ms
                           Asset      Size  Chunks             Chunk Names
      ./0.6168aaac8461862eab7a.js  22.5 kB       0  [emitted]
   ./main.a2e49a279552980e3b91.js    60 kB       1  [emitted]  main
 ./vendor.ff9f7ea865884e6a84c8.js    46 kB       2  [emitted]  vendor
./runtime.25f5d0204e4f77fa57a1.js  1.45 kB       3  [emitted]  runtime
```

&nbsp;&nbsp;&nbsp;↓ 看下面

``` js
[3IRH] ./index.js 29 kB {1} [built]
[DuR2] (webpack)/buildin/global.js 488 bytes {2} [built]
[JkW7] (webpack)/buildin/module.js 495 bytes {2} [built]
[LbCc] ./webPlayer.js 24 kB {1} [built]
[lebJ] ./comments.js 58 kB {0} [built]
[02Tr] ./ads.js 74 kB {1} [built]
    + 1 hidden module
```

With this approach, the ID of a module only changes if you rename or move that module. New modules won’t affect other modules’ IDs.

使用这个方法，只有当重命名或移动该模块时，模块的 ID 才会更改。新的模块也不会影响到其他模块的 ID。

To enable the plugin, add it to the `plugins` section of the config:

可以在配置中的 `plugins` 部分开启这个插件：

``` js
// webpack.config.js
module.exports = {
  plugins: [
    new webpack.HashedModuleIdsPlugin(),
  ],
};
```
### Further reading

### 扩展阅读

* Webpack docs [about the HashedModuleIdsPlugin](https://webpack.js.org/plugins/hashed-module-ids-plugin/)

* Webpack 文档 [关于 HashedModuleIdsPlugin](https://webpack.js.org/plugins/hashed-module-ids-plugin/)

## Summing up

## 总结

* Cache the bundle and differentiate between versions by changing the bundle name

* 缓存 bundle 并通过更改 bundle 名称来进行版本控制

* Split the bundle into app code, vendor code and runtime

* 将 bundle 拆分成 app（应用） 代码、vendor（第三方库） 代码和运行时

* Inline the runtime to save an HTTP request

* 内联运行时可以节省 HTTP 请求

* Lazy-load non-critical code with <code>import</code>

* 使用 import 懒加载非关键代码

* Split code by routes/pages to avoid loading unnecessary stuff

* 按路由或页面拆分代码，从而避免加载不必要的文件
