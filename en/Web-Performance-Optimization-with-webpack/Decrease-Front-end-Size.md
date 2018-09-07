
# 减小前端大小

> - **原文地址：** [decrease frontend size](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size)
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** [减小前端大小](https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Decrease-Front-end-Size.md)
> - **译者：** [杨建](https://github.com/ASkyBig)
> - **校对者：** [泥坤](https://github.com/nkplus)、[周文康](https://github.com/wenkangzhou)

当你正在优化一个应用时，第一件事就是尽可能地减少它的大小。这里介绍如何利用 webpack 来实现。

## 使用生产模式(只在 webpack4)

Webpack 4 引入了 [新的 `mode` 标志](https://webpack.js.org/concepts/mode/). 你可以将这个标志设置为 `'development'` 或者 `'production'` 来指示 webpack 你正在为特定环境构建应用：

```js
// webpack.config.js
module.exports = {
  mode: 'production',
};
```

当为生产环境构建应用时，请确保你设置的模式是 `production` 。 这将让 webpack 应用例如缩小尺寸、移除库中只在开发者模式才有的代码以及[更多](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)的优化。

### 扩展阅读

* [`mode` 标志具体在配置些什么](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)

## 启用缩减

> ⭐️ **注意：** 这些大部分只适用于 webpack 3. 如果你使用 [production 模式下的 webpack4](#enable-the-production-mode), bundle-level 缩减已经启用 – 你只需要启用 [loader-specific 选项](#loader-specific-options).

缩减尺寸是在你通过移除多余的空格、缩短变量的命名等方式压缩代码的时候。例如这样：

``` js
// 原来的代码
function map(array, iteratee) {
  let index = -1;
  const length = array == null ? 0 : array.length;
  const result = new Array(length);

  while (++index < length) {
    result[index] = iteratee(array[index], index, array);
  }
  return result;
}
```

↓

``` js
// 缩减后的代码
function map(n,r){let t=-1;for(const a=null==n?0:n.length,l=Array(a);++t<a;)l[t]=r(n[t],t,n);return l} 
```

Webpack 支持两种方式缩减代码： **bundle-level 缩减** 和 **loader-specific 选项**。 它们应该同步使用。

### Bundle-level 缩减

当编辑完成后，bundle-level 缩减会压缩整个的 bundle 。这里展示了它是如何工作的： 

1.  你的代码是这样的:
    ``` js
    // comments.js
    import './comments.css';
    export function render(data, target) {
      console.log('Rendered!');
    }
    ```
2.  Webpack 大致会将其编译成如下内容:
    
    ``` js
    // bundle.js (part of)
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony export (immutable) */ __webpack_exports__["render"] = render;
    /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__comments_css__ = __webpack_require__(1);
    /* harmony import */ var __WEBPACK_IMPORTED_MODULE_0__comments_css_js___default =
    __webpack_require__.n(__WEBPACK_IMPORTED_MODULE_0__comments_css__);
    
    function render(data, target) {
      console.log('Rendered!');
    }
    ```   
3.  minifier 大致会压缩成下面那样：
    
    ```js
    // minified bundle.js (part of)
    "use strict";function t(e,n){console.log("Rendered!")}
    Object.defineProperty(n,"__esModule",{value:!0}),n.render=t;var o=r(1);r.n(o)
    ```   

**在 webpack 4,** bundle-level 自动启用缩减 – 无论是否在生产模式。它在底层使用的是 [UglifyJS minifier](https://github.com/mishoo/UglifyJS2)。（如果你需要禁用缩减，只要使用开发模式或者将 `optimization.minimize` 选项设置为`false` 。）

**在 webpack 3,** 你需要直接使用 [UglifyJS 插件](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)。这个插件是 webpack 自带的；将它添加到配置的 `plugins` 部分即可启用：

``` js
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
  ],
};  
```

> ⭐️ **注意：** 在 webpack 3 中, UglifyJS 插件不能编译版本超过 ES2015 (即 ES6) 的代码。这意味着如果你的代码使用了类、箭头函数或者其它新的语言特性，你不能将它们编译成 ES5 版本的代码, 否则插件将抛出一个错误。
  
> 如果你需要编译包含新的语法（的代码），使用 [uglifyjs-webpack-plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin) 插件。 这同样是 webpack 自带的插件，但是更新，并且可以编译 ES2015+ 的代码。

### Loader-specific 选项

缩减代码的第二种方法是 loader-specific 选项 ([加载器是什么](https://webpack.js.org/concepts/loaders/)). 利用加载器选项，你可以压缩 minifier 不能缩减的东西。例如，当你利用 [`css-loader`](https://github.com/webpack-contrib/css-loader) 导入一个 CSS 文件时，该文件会被编译成一个字符串：

``` css
/* comments.css */  
.comment {  
  color: black;  
}  
```

↓

``` js
// 缩减后的 bundle.js (部分代码)
exports=module.exports=__webpack_require__(1)(),
exports.push([module.i,".comment {\r\n  color: black;\r\n}",""]);
```

Minifier 不能压缩该代码，因为它是一个字符串。为了缩减文件内容，我们需要像这样配置加载器：

``` js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          { loader: 'css-loader', options: { minimize: true } },
        ],
      },
    ],
  },
};
```

### 扩展阅读

* [UglifyJsPlugin 文档](https://github.com/webpack-contrib/uglifyjs-webpack-plugin)
* 其它流行的 minifier: [Babel Minify](https://github.com/webpack-contrib/babel-minify-webpack-plugin), [Google Closure Compiler](https://github.com/roman01la/webpack-closure-compiler) 

## 指定 **NODE_ENV=production**

> ⭐️ **注意：** 这只适用于 webpack 3. 如果你在[production 模式下使用 webpack 4](#enable-the-production-mode),  `NODE_ENV=production` 优化已启用 – 可自由选择地跳过该部分.

减少前端大小的另一种方法在你的代码中将 `NODE_ENV` [环境变量](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them) 设置为 `production`.

库会读取 `NODE_ENV` 变量以检测它们应该在哪个模式下工作 – 在开发或生产中。 有些库基于该变量而有不同的表现。例如，当 `NODE_ENV` 没有设置为 `production`, Vue.js 会做额外的检查并打印警告：

``` js
// vue/dist/vue.runtime.esm.js
// …
if (process.env.NODE_ENV !== 'production') {
  warn('props must be strings when using array syntax.');
}
// … 
```

React 表现类似 – 它加载包含警告的开发环境构建：

``` js
// react/index.js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./cjs/react.production.min.js');
} else {
  module.exports = require('./cjs/react.development.js');
}

// react/cjs/react.development.js
// …
warning$3(
  componentClass.getDefaultProps.isReactClassApproved,
  'getDefaultProps is only used on classic React.createClass ' +
  'definitions. Use a static property named `defaultProps` instead.'
);
// … 
```

生产中通常不需要这些检查和警告，但是它们还是存在于代码中并增加了库的大小。 **在 webpack 4,** 通过添加 `optimization.nodeEnv: 'production'` 选项以移除它们:

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    nodeEnv: 'production',
    minimize: true,
  },
}; 
```

**在 webpack 3 中，** 使用 [`DefinePlugin`](https://webpack.js.org/plugins/define-plugin/) 替代:

```
// webpack.config.js (for webpack 3)
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"',
    }),
    new webpack.optimize.UglifyJsPlugin(),
  ],
}; 
```

`optimization.nodeEnv` 选项和 `DefinePlugin` 工作方式相同 – 它们会用某个特定的值取代所有在执行的 process.env.NODE_ENV。使用上面的配置：

1.  Webpack 会将所有存在的 `process.env.NODE_ENV` 替换成 `"production"`:
    ``` js
    // vue/dist/vue.runtime.esm.js
    if (typeof val === 'string') {
      name = camelize(val);
      res[name] = { type: null };
    } else if (process.env.NODE_ENV !== 'production') {
      warn('props must be strings when using array syntax.');
    }
    ```
    ↓
    ``` js
    // vue/dist/vue.runtime.esm.js
    if (typeof val === 'string') {
      name = camelize(val);
      res[name] = { type: null };
    } else if ("production" !== 'production') {
      warn('props must be strings when using array syntax.');
    }
    ```
2.  然后 [minifier](#enable-minification) 将会移除所有像 `if` 这样的分支 – 因为 `"production" !== 'production'` 总是错误的，插件明白这些分支中的代码永远不会执行：
    
    ``` js
    // vue/dist/vue.runtime.esm.js
    if (typeof val === 'string') {
      name = camelize(val);
      res[name] = { type: null };
    } else if ("production" !== 'production') {
      warn('props must be strings when using array syntax.');
    }
    ```
    ↓
    ``` js
    // vue/dist/vue.runtime.esm.js (without minification)
    if (typeof val === 'string') {
      name = camelize(val);
      res[name] = { type: null };
    }
    ```
    
### 扩展阅读

* [“环境变量”是什么](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them)
* 关于: [`DefinePlugin`](https://webpack.js.org/plugins/define-plugin/), [`EnvironmentPlugin`](https://webpack.js.org/plugins/environment-plugin/) 的 Webpack 文档

## 使用 ES 模块

降低前端尺寸的另一种方法是使用 [ES 模块](https://ponyfoo.com/articles/es6-modules-in-depth).

当你使用 ES 模块, webpack 就可以进行 tree-shaking。Tree-shaking 是当 bundler 遍历整个依赖树时，检查使用了什么依赖，并移除无用的。所以，如果你使用了 ES 模块语法， webpack 可以去掉未使用的代码：

1.  你写了一个带有多个 export 的文件，但是应用只使用它们其中的一个：
    
    ``` js
    // comments.js
    export const render = () => { return 'Rendered!'; };
    export const commentRestEndpoint = '/rest/comments';
    
    // index.js
    import { render } from './comments.js';
    render();
    ```
2.  Webpack 明白 `commentRestEndpoint` 没有用到并且不会在 bundle 中生成单独的导出点： 
    
    ``` js
    // bundle.js (part that corresponds to comments.js)
    (function(module, __webpack_exports__, __webpack_require__) {
      "use strict";
      const render = () => { return 'Rendered!'; };
      /* harmony export (immutable) */ __webpack_exports__["a"] = render;
    
      const commentRestEndpoint = '/rest/comments';
      /* unused harmony export commentRestEndpoint */
    })
    ```
    
3.  [minifier](#enable-minification) 移除未使用的变量：
     ``` js
    // bundle.js (part that corresponds to comments.js)
    (function(n,e){"use strict";var r=function(){return"Rendered!"};e.b=r})
     ```

This works even with libraries if they are written with ES modules.

> ⭐️ **注意:** 在 webpack 中, tree-shaking 没有 minifier 是不会起作用的。Webpack 仅移除导出语句中没有使用到的；是 minifier 移除未使用的代码的。所以，如果你在没有 minifier 的情况下编译 bundle，是不会减小的。
  
> 然而，你不需要特定使用 webpack 内置的 minifier (`UglifyJsPlugin`). 任意的 minifier 都支持移除无用代码(例如 [Babel Minify plugin](https://github.com/webpack-contrib/babel-minify-webpack-plugin) 或 [Google Closure Compiler plugin](https://github.com/roman01la/webpack-closure-compiler)) 都可以奏效.

> ❗ **警告:** 不要意外地将 ES 模块编译为 CommonJS 模块。
  
> 如果你使用 Babel 的 `babel-preset-env` 或 `babel-preset-es2015`， 检查它们预先的设置。默认情况下， 它们将 ES 的 `import` 和 `export` 转译为 CommonJS 的 `require` 和 `module.exports`. [通过 `{ modules: false }` 选项](https://github.com/babel/babel/tree/master/packages/babel-preset-env) 来禁用它.  
  
> 与 TypeScript 相同：记得在你的 `tsconfig.json` 中设置 `{ "compilerOptions": { "module": "es2015" } }`。

### 扩展阅读

* [“深入 ES6 模块”](https://ponyfoo.com/articles/es6-modules-in-depth) 
* Webpack 文档 [关于 tree shaking](https://webpack.js.org/guides/tree-shaking/)  

## 优化图片

图片占页面大小的[一半以上](http://httparchive.org/interesting.php?a=All&l=Oct%2016%202017)。 尽管它们不如 JavaScript 关键(例如，它们不会阻塞渲染)，它们仍然消耗了带宽的一大部分。在 webpack 中使用 `url-loader`, `svg-url-loader` 和 `image-webpack-loader` 来优化它们。

[`url-loader`](https://github.com/webpack-contrib/url-loader) 将小的静态文件内联进应用。没有配置的话，它需要通过传递文件，将它放在编译后的打包 bundle 内并返回一个这个文件的 url。然而，如果我们指定了 `limit` 选项，它会将文件编码成比无配置更小的 [Base64 的数据 url](https://css-tricks.com/data-uris/) 并将该 url 返回。这样可以将图片内联进 JavaScript 代码中，并节省一次 HTTP 请求：

``` js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif)$/,
        loader: 'url-loader',
        options: {
          // Inline files smaller than 10 kB (10240 bytes)
          limit: 10 * 1024,
        },
      },
    ],
  }
};
```

``` js
// index.js
import imageUrl from './image.png';
// → If image.png is smaller than 10 kB, `imageUrl` will include
// the encoded image: 'data:image/png;base64,iVBORw0KGg…'
// → If image.png is larger than 10 kB, the loader will create a new file,
// and `imageUrl` will include its url: `/2fcd56a1920be.png`
```

> ⭐️ **注意：** 内联图片减少了单独请求的数量，这是好的([即使通过 HTTP/2](https://blog.octo.com/en/http2-arrives-but-sprite-sets-aint-no-dead/))，但是增加了 bundle 和内存消耗的下载/解析时间。确保不要嵌入大的或者很多的图片，否则增加的 bundle 时间可能超过内联带来的好处。

[`svg-url-loader`](https://github.com/bhovhannes/svg-url-loader)的工作原理类似于 `url-loader` – 除了它利用 [URL encoding](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding) 而不是 Base64 对文件编码。对于 SVG 图片这是有效的 – 因为 SVG 文件恰好是纯文本，这种编码规模效应更加明显：

``` js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.svg$/,
        loader: 'svg-url-loader',
        options: {
          // Inline files smaller than 10 kB (10240 bytes)
          limit: 10 * 1024,
          // Remove the quotes from the url
          // (they’re unnecessary in most cases)
          noquotes: true,
        },
      },
    ],
  },
};
```

> ⭐️ **注意:** svg-url-loader 拥有改善 IE 浏览器支持的选项，但是在其他浏览器中更糟糕。如果你需要兼容 IE 浏览器，[设置 `iesafe: true` 选项](https://github.com/bhovhannes/svg-url-loader#iesafe).

[`image-webpack-loader`](https://github.com/tcoopman/image-webpack-loader) 会压缩检查到的所有图片。它支持 JPG、PNG、GIF 和 SVG 格式的图片，因此我们在碰到所有这些类型的图片都会使用它。

此加载器不能将图片嵌入应用，所以它必须和 `url-loader` 以及 `svg-url-loader` 一起使用。为了避免同时将它复制粘贴到两个规则中（一个针对 JPG/PNG/GIF 图片， 另一个针对 SVG ），我们使用 [`enforce: 'pre'`](https://webpack.js.org/configuration/module/#rule-enforce) 作为单独的规则涵盖在这个加载器：

``` js
 // webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(jpe?g|png|gif|svg)$/,
        loader: 'image-webpack-loader',
        // This will apply the loader before the other ones
        enforce: 'pre',
      },
    ],
  },
};
```

加载器的默认设置已经很好了 - 但是如果你想更进一步去配置它，参考[插件选项](https://github.com/tcoopman/image-webpack-loader#options). 要选择指定选项，请查看 Addy Osmani 的[图像优化指南](https://images.guide/).

### 扩展阅读

* ["base64 编码用于什么？"](https://stackoverflow.com/questions/201479/what-is-base-64-encoding-used-for)   
* Addy Osmani 的[图像优化指南](https://images.guide/)
    
## 优化依赖

平均一半以上的 Javascript 体积大小来源于依赖包，并且这其中的一部分可能都不是必要的。

例如，Lodash (自 v4.17.4 版本起) 向 bundle 增加了 72KB 的缩小代码。但是如果你仅使用它差不多20种的方法，那么大约 65KB 的缩小代码什么都不做。

另一个例子是 Moment.js。它的 2.19.1 版本占据了 223KB 的缩减代码，这是巨大的 - 截至 2017 年 10 月，一个页面的 JavaScript 平均体积是 [452 KB] (http://httparchive.org/interesting.php?a=All&l=Oct%2016%202017)。然而，其中的 170KB 是[本地化文件](https://github.com/moment/moment/tree/4caa268356434f3ae9b5041985d62a0e8c246c78/locale)。如果你没有用到多语言版 Moment.js，这些文件都将毫无目的地使 bundle 更臃肿。

所有的这些依赖都可以轻易地优化。我们已经在 GitHub 仓库中收集了优化方法 - [检查出来](https://github.com/GoogleChromeLabs/webpack-libs-optimizations)!

## 为 ES 模块启用模块串联（又称作用域提升）

> ⭐️ **注意：** 如果在生产模式下使用 [webpack 4](#启用生产模式),模块串联已经启用。自由地跳过该部分。

当你构建 bundle 时，webpack 将每个模块包装进一个函数中：

``` js
// index.js
import {render} from './comments.js';
render();

// comments.js
export function render(data, target) {
  console.log('Rendered!');
}
```

↓

``` js
// bundle.js (part  of)
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {

  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
  var __WEBPACK_IMPORTED_MODULE_0__comments_js__ = __webpack_require__(1);
  Object(__WEBPACK_IMPORTED_MODULE_0__comments_js__["a" /* render */])();

}),
/* 1 */
(function(module, __webpack_exports__, __webpack_require__) {

  "use strict";
  __webpack_exports__["a"] = render;
  function render(data, target) {
    console.log('Rendered!');
  }

})
```

过去，需要将 CommonJS/AMD 模块相互隔离。然而，这增加了每个模块的大小和性能开支。

Webpack 2 引入了对 ES 模块的支持，不同于 CommonJS 和 AMD 模块，可以不需要通过一个方法包装而捆绑在一起。并且 webpack 3 使这样的捆绑变得可能 -  通过 [模块连接](https://webpack.js.org/plugins/module-concatenation-plugin/)。这是模块连接的工作原理：

``` js
// index.js
import {render} from './comments.js';
render();

// comments.js
export function render(data, target) {
  console.log('Rendered!');
}
```

↓

``` js
// Unlike the previous snippet, this bundle has only one module
// which includes the code from both files

// bundle.js (part of; compiled with ModuleConcatenationPlugin)
/* 0 */
(function(module, __webpack_exports__, __webpack_require__) {

  "use strict";
  Object.defineProperty(__webpack_exports__, "__esModule", { value: true });

  // CONCATENATED MODULE: ./comments.js
  function render(data, target) {
    console.log('Rendered!');
  }

  // CONCATENATED MODULE: ./index.js
  render();

})
```

看到不同了吗？在普通绑定中，模块 0 需要模块 1 的 `render`。使用模块连接， `require` 只需用所需要的功能替换，模块 1 就被移除了。 bundle 拥有更小的模块 – 以及更少的模块开支!

要在 **webpack 4** 中开启这个功能，启用 `optimization.concatenateModules` 选项即可：

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    concatenateModules: true,
  },
};
```

在 **webpack 3** 中，使用 `ModuleConcatenationPlugin` 插件:

``` js
// webpack.config.js (for webpack 3)
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin(),
  ],
};
```

> ⭐️ **注意：** 想知道为什么默认不启用这个行为吗？连接模块是很棒， [但是它增加了构建时间并破坏了模块热替换](https://twitter.com/TheLarkInn/status/925800563144454144)。这是为什么它只在生产下启用。

### 扩展阅读

* Webpack 文档 [用于 ModuleConcatenationPlugin](https://webpack.js.org/plugins/module-concatenation-plugin/)   
* [”作用域提升简介“](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f)  
* [此插件作用的](https://medium.com/webpack/webpack-freelancing-log-book-week-5-7-4764be3266f5) 详细描述

## 使用 `externals` ，如果同时包含 webpack 和非 webpack 代码

你可能有一个大的项目，其中有些代码是用 webpack 编译的，有些不是。类似于视频托管网站，播放器小部件可能是 webpack 构建的，而周围的页面不是：

![视频托管网站屏幕快照](https://developers.google.com/web/fundamentals/performance/webpack/video-hosting.png)

(完全随机的视频托管网站)

如果代码块有公共的依赖，你可以共享它们以避免多次下载其代码。这是通过 [webpack 的 `externals` 选项]完成的(https://webpack.js.org/configuration/externals/) – 它通过变量或其它的额外导入来替换模块。

### 如果依赖在 `window` 中可获得

如果你的 non-webpack 代码依赖于某些依赖，这些依赖在 `window` 中可以作为变量获得，将依赖名别名为变量名：

``` js
// webpack.config.js
module.exports = {
  externals: {
    'react': 'React',
    'react-dom': 'ReactDOM',
  },
};
```

通过这个配置， webpack 不会打包 `react` 和 `react-dom` 包。相反，它们将被替换成下面这样的东西：

``` js
// bundle.js (part of)
(function(module, exports) {
  // A module that exports `window.React`. Without `externals`,
  // this module would include the whole React bundle
  module.exports = React;
}),
(function(module, exports) {
  // A module that exports `window.ReactDOM`. Without `externals`,
  // this module would include the whole ReactDOM bundle
  module.exports = ReactDOM;
})
```

### 如果依赖是当做 AMD 包被加载的情况

如果你的 non-webpack 代码没有将依赖暴露于 `window`，事情就变得更加复杂。然而，如果这些非 webpack 代码将这些依赖作为 [AMD 包](http://requirejs.org/docs/whyamd.html#amd)，你仍然可以避免相同的代码加载两次。

具体如何做呢，将 webpack 代码编译成一个 AMD bundle ，同时将模块别名为库的 URLs：

``` js
// webpack.config.js
module.exports = {
  output: { libraryTarget: 'amd' },

  externals: {
    'react': { amd: '/libraries/react.min.js' },
    'react-dom': { amd: '/libraries/react-dom.min.js' },
  },
};
```

Webpack 将把 bundle 包装进 `define()` 并让其依赖于这些 URLs：

``` js
// bundle.js (beginning)
define(["/libraries/react.min.js", "/libraries/react-dom.min.js"], function () { … });
```

如果 non-webpack 代码使用了相同的 URLs 来加载它的依赖，那么这些文件只会加载一次 - 额外的请求将使用加载器缓存。

> ⭐️ **注意：** Webpack 仅替换那些明确匹配 `externals` 对象的键的导入。这意味着如果你编写 `import React from 'react/umd/react.production.min.js'`，这个库不会从 bundle 中排除。这是合理的 - webpack 不知道 `import 'react'` 和 `import 'react/umd/react.production.min.js'` 是否是同一个东西 - 所以保持小心。

### 扩展阅读

* Webpack 文档 [on `externals`](https://webpack.js.org/configuration/externals/)

## 总结

* 如果使用 webpack 4，请启用生产模式
* 缩减你的代码，通过 bundle-level minifier 和 loader 选项
* 移除仅在开发环境使用的代码，通过将 `NODE_ENV` 替换成 `production`
* 使用 ES 模块以启用 tree shaking
* 压缩图片
* 启用特定依赖优化
* 启用模块连接
* 使用 `externals`，如果这对你有效果
