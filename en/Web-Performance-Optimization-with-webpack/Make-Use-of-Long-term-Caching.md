# 利用好持久化缓存

> - **原文地址：** [use long term caching](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching)
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** [利用好持久化缓存](https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Make-Use-of-Long-term-Caching.md)
> - **译者：** [周文康](https://github.com/wenkangzhou)
> - **校对者：** [闫蒙](https://github.com/yanyixin)、[泥坤](https://github.com/nkplus)

在[优化应用体积](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size)之后，下一个提升应用加载时间的策略就是缓存。将资源缓存在客户端中，可以避免之后每次都重新下载。

## bundle 的版本控制和缓存头的使用

使用缓存的通用方法：

 1. 告诉浏览器需要缓存一个文件很长时间（比如，一年）

    ``` js
    # Server header
    Cache-Control: max-age=31536000
    ```
    
    注意：如果你不熟悉 `Cache-Control` 的原理，请参阅 Jake Archibald 的文章: [关于缓存的最佳实践](https://jakearchibald.com/2016/caching-best-practices/)。

2. 当文件改变时，文件会被重命名，这样就迫使浏览器重新下载：

    ``` js
    <!-- 修改前 -->
    <script src="./index-v15.js"></script>
    
    <!-- 修改后 -->
    <script src="./index-v16.js"></script>
    
    ```

这个方法可以告诉浏览器去下载 JS 文件，并将它缓存，之后使用的都是它的缓存副本。浏览器只会在文件名发生改变（或者一年之后缓存失效）时才会请求网络。

使用 webpack，同样可以做到，但使用的不是版本号，而是指定文件的哈希值。使用 [`[chunkhash]`](https://webpack.js.org/configuration/output/#output-filename) 可以将哈希值写入文件名中：
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

> ⭐️ 注意: 即使 bundle 不变，webpack 也可能生成不同的哈希值 – 例如，你重命名了一个文件或者在不同的操作系统下编译了 bundle。 当然，这其实是一个  bug，目前还没有明确的解决方案，[具体可参阅 GitHub 上的讨论](https://github.com/webpack/webpack/issues/1479)。

如果你需要将文件名发送给客户端，可以使用 `HtmlWebpackPlugin` 或者 `WebpackManifestPlugin`。

[`HtmlWebpackPlugin`](https://github.com/jantimon/html-webpack-plugin) 是一个简单但扩展性不强的插件。在编译期间，它会生成一个 HTML 文件，文件包含了所有已经被编译的资源。如果你的服务端逻辑不是很复杂，那么它应该能满足你：

```js
<!-- index.html -->
<!doctype html>
<!-- ... -->
<script src="bundle.8e0d62a03.js"></script>
```

[`WebpackManifestPlugin`](https://github.com/danethurber/webpack-manifest-plugin) 是一个扩展性更佳的插件，它可以帮助你解决服务端逻辑比较复杂的那部分。在打包时，它会生成一个 JSON 文件，里面包含了原文件名和带哈希文件名的映射。在服务端，通过这个 JSON 就能方便的找到我们真正要执行的文件：

``` js
// manifest.json
{
  "bundle.js": "bundle.8e0d62a03.js"
}
```

### 扩展阅读

* Jake Archibald [关于缓存的最佳实践](https://jakearchibald.com/2016/caching-best-practices/)

## 将依赖和 runtime 提取到单独的文件中

### 依赖

应用的依赖通常比实际应用内的代码变更频率低。如果将它们移到单独的文件中，浏览器就可以独立缓存它们 – 这样每次应用中的代码变更也不会去重新下载它们。

> 关键术语：在 webpack 术语中，把带有应用代码的独立文件称之为 **chunk**。我们在下面的文章中会使用到这个名称。

要将依赖项提取到独立的 chunk 中，需要执行下面三个步骤：

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

  当 webpack 编译应用时，它会将[`[name]`](https://webpack.js.org/configuration/output/#output-filename) 作为 chunk 的名称。如果我们没有添加 `[name]` 的部分，我们将不得不通过哈希值来区分 chunk - 这样就变得非常困难！

2. 将 `entry` 的值改为对象：
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
    
    在上面这段代码中，“main” 是 chunk 的名称。这个名称会在第一步时被 `[name]` 所替代。
    
    到目前为止，如果你构建应用，这个 chunk 还是包含了整个应用的代码 - 就像我们没有做过上述这些步骤一样。但接下来很快就将产生变化。

3. **在 webpack 4 中**，可以将 `optimization.splitChunks.chunks: 'all'` 选项添加到 webpack 的配置中:

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
    
    这个选项可以开启智能代码拆分。使用了这个功能，webpack 将会提取大于 30KB（压缩和 gzip 之前）的第三方库代码。它同时也可以提取公共代码 - 如果你的构建结果会生成多个 bundle 时这将非常有用。（例如：[假如你通过路由来拆分应用](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching#split-the-code-into-routes-and-pages)）。
     
    **在 webpack 3 中**添加 [CommonsChunkPlugin](https://webpack.js.org/plugins/commons-chunk-plugin/) 插件:
    
    ``` js
    // webpack.config.js (for webpack 3)
    module.exports = {
      plugins: [
        new webpack.optimize.CommonsChunkPlugin({
          // chunk 的名称将会包含依赖
          // 这个名称会在第一步时被 [name] 所替代
          name: 'vendor',
    
          // 这个函数决定哪个模块会被打入 chunk
          minChunks: module => module.context &&
            module.context.includes('node_modules'),
        }),
      ],
    };
    ```
   
    这个插件会将路径包含 `node_modules` 的所有模块移到一个名为 vendor.[chunkhash].js 的独立文件中。

完成这些更改后，每次打包都将从原来的生成一个文件变为生成两个文件：`main.[chunkhash].js` 和`vendor.[chunkhash].js` (`vendors~main.[chunkhash].js` 只有在 webpack 4 才有)。在 webpack 4 中，如果依赖项很小，则可能不会生成 vendor bundle - 这点做的不错：

``` js
$ webpack
Hash: ac01483e8fec1fa70676
Version: webpack 3.8.1
Time: 3816ms
                           Asset   Size  Chunks             Chunk Names
  ./main.00bab6fd3100008a42b0.js  82 kB       0  [emitted]  main
./vendor.d9e134771799ecdf9483.js  47 kB       1  [emitted]  vendor
```

浏览器会单独缓存这些文件 - 同时只有代码发生改变时才会重新下载。

### Webpack runtime 代码

遗憾的是，仅仅提取第三方库代码还是不够的。如果你想尝试在应用代码中修改一些东西：

``` js
// index.js
…
…

// 例如，增加这句:
console.log('Wat');
```

你会发现 `vendor` 的哈希值也会被改变：

``` js
                           Asset   Size  Chunks             Chunk Names
./vendor.d9e134771799ecdf9483.js  47 kB       1  [emitted]  vendor
```

↓

``` js
                            Asset   Size  Chunks             Chunk Names
./vendor.e6ea4504d61a1cc1c60b.js  47 kB       1  [emitted]  vendor
```

这是由于 webpack 打包时，除了模块代码之外，webpack 的 bundle 中还包含了 **[runtime](https://webpack.js.org/concepts/manifest/)**  - 一小段可以管理模块执行的代码。当你将代码拆分成多个文件时，这小部分代码在 chunk id 和匹配的文件之间会生成一个映射：

``` js
// vendor.e6ea4504d61a1cc1c60b.js
script.src = __webpack_require__.p + chunkId + "." + {
  "0": "2f2269c7f0a55a5c1871"
}[chunkId] + ".js";
```

Webpack 将 runtime 包含在了最新生成的 chunk 中，这个 chunk 就是我们代码中的 `vendor`。每次 chunk 有任何变更，这一小部分代码也会随之更改，同时也会导致整个 `vendor` chunk 发生改变。

为了解决这个问题，我们可以将 runtime 移动到一个独立的文件中。**在 webpack 4 中**，可以通过开启 `optimization.runtimeChunk` 选项来实现：

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    runtimeChunk: true,
  },
};
```

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

    // 这个插件必须在 vendor 生成之后执行（因为 webpack 把运行时打进了最新的 chunk）
    new webpack.optimize.CommonsChunkPlugin({
      name: 'runtime',

      // minChunks: Infinity 表示任何应用模块都不能打进这个 chunk
      minChunks: Infinity,
    }),
  ],
};
```

完成这些变更后，每次构建将生成三个文件：

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

将这几个文件按倒序的方式添加到 `index.html` 中，就完成了：

``` js
<!-- index.html -->
<script src="./runtime.79f17c27b335abc7aaf4.js"></script>
<script src="./vendor.26886caf15818fa82dfa.js"></script>
<script src="./main.00bab6fd3100008a42b0.js"></script>
```

### 扩展阅读

* Webpack 指南 [关于持久化缓存](https://webpack.js.org/guides/caching/)
  
* Webpack 文档 [关于 webpack 的 runtime 和 manifest 文件](https://webpack.js.org/concepts/manifest/)

* [“CommonsChunkPlugin 的最佳实践”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)

* [`optimization.splitChunks` 和 `optimization.runtimeChunk` 的工作原理](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

## 内联 webpack 的 runtime 可以节省额外的 HTTP 请求

为了达到更好的体验，我们可以尝试把 webpack 的 runtime 内联到 HTML 中。例如，我们不要这么做：

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

Runtime 的代码不多，内联到 HTML 中可以帮助我们节省 HTTP 请求（在 HTTP/1 中尤为重要；在 HTTP/2 中虽然没那么重要，但仍然能起到一定作用）。

下面就来看看要如何做。

### 如果你使用 HtmlWebpackPlugin 来生成 HTML

如果你使用 [HtmlWebpackPlugin](https://github.com/jantimon/html-webpack-plugin) 来生成 HTML 文件，那么你一定需要 [InlineSourcePlugin](https://github.com/DustinJackson/html-webpack-inline-source-plugin) ：

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

### 如果你通过自定义服务端逻辑来生成 HTML

**在 webpack 4 中:**

1. 添加 [WebpackManifestPlugin](https://github.com/danethurber/webpack-manifest-plugin) 插件可以获取生成的 runtume chunk 的名称：

    ``` js
    // webpack.config.js (for webpack 4)
    const ManifestPlugin = require('webpack-manifest-plugin');
    
    module.exports = {
      plugins: [
        new ManifestPlugin(),
      ],
    };
    ```
    
    使用这个插件构建会生成像下面这样的文件：
    
    ``` js
    // manifest.json
    {
      "runtime~main.js": "runtime~main.8e0d62a03.js"
    }
    ```

2. 可以用一个便利的方式内联 runtime chunk 的内容。例如，使用 Node.js 和 Express：

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

**在 webpack 3 中:**

1. 通过指定 `filename` ，可以使 runtime 的名称不发生改变 :

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

2. 可以用一个便利的方式内联 <code>runtime.js</code> 的内容。例如，使用 Node.js 和 Express：

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

## 代码懒加载

通常，一个网页会有自身的侧重点：
  
* 假如你在 YouTube 上加载一个视频页面，你更关心的肯定是视频而不是评论. 所以，这里视频就比评论重要。

* 又比如你在一个新闻网站看一篇文章，你更关心的肯定是文章的文字而不是广告. 所以，这里文字就比广告重要。

上面的这些情况，都可以通过优先下载最重要的部分，稍后懒加载剩余部分，从而来提升页面首次加载的性能。在 webpack 中，使用[`import()` 函数](https://webpack.js.org/api/module-methods/#import-) 和[代码拆分](https://webpack.js.org/guides/code-splitting/)即可实现。

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

`import()` 函数可以帮助你实现按需加载。Webpack 在打包时遇到 `import('./module.js')`，就会把这个模块放到单独的 chunk 中：


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

只有当代码执行到 `import()` 函数时才会去下载。

这样可以让 `入口` bundle 变得更小，从而减少首次加载时间。不仅如此，它还可以优化缓存 - 如果你修改了入口 chunk 的代码，注释 chunk 不会受到影响。

> ⭐️ 注意: 如果你使用 Babel 编译代码，会因为 Babel 无法识别 `import()` 而出现语法错误。为了避免这个错误，你可以添加 [`syntax-dynamic-import`](https://www.npmjs.com/package/babel-plugin-syntax-dynamic-import) 插件。

### 扩展阅读

* Webpack 文档 [`import()` 函数的使用](https://webpack.js.org/api/module-methods/#import-)

* JavaScript 提案  [实现 `import()` 语法](https://github.com/tc39/proposal-dynamic-import)

## 将代码拆分为路由和页面

如果你的应用有多个路由或页面，但是代码中只有一个单独的 JS 文件（一个单独的`入口` chunk），这样似乎会让你的每次请求都附加了额外的流量。例如，当用户访问你网站的首页：

![](https://developers.google.com/web/fundamentals/performance/webpack/site-home-page.png)

他们并不需要加载其它页面上用于渲染文章的代码 - 但他们却加载了。此外，如果这个用户经常只是访问首页，但你更改了其它页面的文章代码，webpack 将会重新编译，使整个 bundle 失效 - 这样将导致用户重新下载整个应用的代码。

如果我们将代码拆分到页面中（或者单页面应用的路由里），用户就会只下载真正用到的那部分代码。此外，浏览器也会更好地缓存应用代码：当你改变首页的代码时，webpack 只会让相匹配的 chunk 失效。

### 单页面应用

要通过路由来拆分单页应用，可以使用 `import()`（参加上文[代码懒加载](#lazy-loading)部分）。如果你使用的是一个框架，目前也有现成的解决方案：

* `react-router` 文档中的["代码分离”](https://reacttraining.com/react-router/web/guides/code-splitting)(适用于 React)

* `vue-router` 文档中的[“路由的懒加载”](https://router.vuejs.org/en/advanced/lazy-loading.html)(适用于 Vue.js)

### 传统多页应用

要通过页面来拆分传统应用，可以使用 webpack 的 [entry points](https://webpack.js.org/concepts/entry-points/)。假设你的应用中有三类页面：主页、文章页和用户账户页，- 那么就应该有三个入口：

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

所以，如果只有 article 页面使用到了 Lodash，那么 home 和 profile bundle 就不会包含它 - 用户也不会在访问首页的时候下载到这个库。

但是，单独的依赖树有它们的缺点。如果两个入口都使用到了 Lodash，同时你没有将依赖项移到 vendor bundle 中，则两个入口都将包含 Lodash 的副本。为了解决这个问题，**在 webpack 4 中**，可以在你的 webpack 配置中加入`optimization.splitChunks.chunks: 'all'`选项：

``` js
// webpack.config.js (适用于webpack 4)
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all',
    }
  },
};
```

这个选项可以开启智能代码拆分。有了这个选项，webpack 将自动查找到公共代码，并且提取到单独的文件中。

**在 webpack 3 中**，可以使用 [`CommonsChunkPlugin`](https://webpack.js.org/plugins/commons-chunk-plugin/) 插件，它会将公共的依赖项移动到一个新的指定文件中：

``` js
// webpack.config.js (适用于 webpack 3)
module.exports = {
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      // chunk 的名称将会包含公共依赖
      name: 'common',

      // minChunks表示要将一个模块打入公共文件时必须包含的 `minChunks` chunks 数量
      // (注意，插件会分析所有 chunks 和 entries)
      minChunks: 2,    // 2 is the default value
    }),
  ],
};
```

你可以尝试调整 `minChunks` 的值来找到最优的方案。通常情况下，你希望它是一个较小的值，但随着 chunk 数量的增加它会随之增大。例如，有 3 个 chunk 时，`minChunks` 的值可能是 2 ，但是有 30 个 chunk 时，它的值可能是 8 - 因为如果你把它设置成 2，就会有很多模块要被打包进同一个公共文件中，这样文件就会变得臃肿。

### 扩展阅读

* Webpack 文档 [关于 entry points 的概念](https://webpack.js.org/concepts/entry-points/)

* Webpack 文档 [关于 CommonsChunkPlugin 插件](https://webpack.js.org/plugins/commons-chunk-plugin/)

* [“CommonsChunkPlugin 的最佳实践”](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)

* [`optimization.splitChunks` 和 `optimization.runtimeChunk` 的工作原理](https://gist.github.com/sokra/1522d586b8e5c0f5072d7565c2bee693)

## 确保模块的 id 更加稳定

构建代码时，webpack 会为每个模块分配一个 ID。随后，这些 ID 将在 bundle 里的 `require()` 函数中被使用到。你通常会在编译输出的模块路径前看到这些 ID：

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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ 我们添加了一个新模块...

``` js
   [4] ./webPlayer.js 24 kB {1} [built]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ 看看下面做了什么！ `comments.js` 的 ID 由 4 变成了 5

``` js
   [5] ./comments.js 58 kB {0} [built]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓ `ads.js` 的 ID 由 5 变成了 6

``` JS
   [6] ./ads.js 74 kB {1} [built]
       + 1 hidden module
``` 

这将使包含或依赖于这些被更改 ID 的模块的所有 chunk 都无效 - 即使它们实际代码没有更改。在我们的案例中，ID 为 `0` 的 chunk ( `comments.js` 的 chunk)  和 `main` chunk （其它应用代码的 chunk ）都将失效 - 但其实只有 `main` 应该失效。

为了解决这个问题，可以使用 [`HashedModuleIdsPlugin`](https://webpack.js.org/plugins/hashed-module-ids-plugin/) 插件来改变模块 ID 的计算方式。这个插件用模块路径的哈希值代替了基于计数器的 ID：

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

使用了这个方法，只有当重命名或移动该模块时，模块的 ID 才会更改。新的模块也不会影响到其他模块的 ID。

可以在配置中的 `plugins` 部分开启这个插件：

``` js
// webpack.config.js
module.exports = {
  plugins: [
    new webpack.HashedModuleIdsPlugin(),
  ],
};
```

### 扩展阅读

* Webpack 文档 [关于 HashedModuleIdsPlugin](https://webpack.js.org/plugins/hashed-module-ids-plugin/)

## 总结

* 缓存 bundle 并通过更改 bundle 名称来进行版本控制

* 将 bundle 拆分成 app（应用） 代码、vendor（第三方库） 代码和 runtime

* 内联 runtime 可以节省 HTTP 请求

* 使用 <code>import</code> 懒加载非关键代码

* 按路由或页面拆分代码，从而避免加载不必要的文件
