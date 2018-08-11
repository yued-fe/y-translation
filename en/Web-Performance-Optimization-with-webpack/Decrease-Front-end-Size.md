
# 减小前端大小

> - **原文地址：** https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Decrease-Front-end-Size.md
> - **译者：** 
> - **校对者：**

当你正在优化一个应用时，第一件事就是尽可能地减少它的大小。这里介绍如何利用webpack来实现。

## 使用生产模式(只在webpack 4)

Webpack 4 介绍了 [新的 `mode` 标志](https://webpack.js.org/concepts/mode/). 你可以将这个标志设置为 `'development'` 或者 `'production'` 来指示 webpack 你正在为特定环境构建应用：

```js
// webpack.config.js
module.exports = {
  mode: 'production',
};
```

当在生产环境构建应用时，请确保你设置的模式是 `production` 。 这将让 webpack 应用例如缩小尺寸、移除代码库中只在开发者模式才有的代码以及[更多](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)的一些优化。

### 扩展阅读

* [What specific things the `mode` flag configures](https://medium.com/webpack/webpack-4-mode-and-optimization-5423a6bc597a)

## 启用缩减尺寸

> ⭐️ **注意:** most of this is webpack 3 only. 如果你使用 [production 模式下的webpack 4](#enable-the-production-mode), bundle-level 缩减已经启用 – 你只需要启用 [特定加载选项](#loader-specific-options).

缩减尺寸是在你通过移除额外的空间、缩短变量的命名等方式压缩代码的时候。例如这样：

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

Webpack 支持两种方式缩减代码： **bundle-level 缩减** and **loader-specific 选项**. 它们应该同步使用。

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
2.  Webpack 大约会将其编译成如下内容:
    
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
3.  A minifier 大约会压缩成下面那样：
    
    ```js
    // minified bundle.js (part of)
    "use strict";function t(e,n){console.log("Rendered!")}
    Object.defineProperty(n,"__esModule",{value:!0}),n.render=t;var o=r(1);r.n(o)
    ```   

**在 webpack 4,** the bundle-level 自动启用缩减 – both in the production mode and without one. 它使用 [the UglifyJS minifier](https://github.com/mishoo/UglifyJS2) under the hood. (If you ever need to disable minification, just use the development mode or pass `false` to the `optimization.minimize` option.)

**在 webpack 3,** 你需要直接使用 [UglifyJS 插件](https://github.com/webpack-contrib/uglifyjs-webpack-plugin) . The plugin comes bundled with webpack; to enable it, add it to the `plugins` section of the config:

``` js
// webpack.config.js
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
  ],
};  
```

> ⭐️ **注意:** 在 webpack 3, UglifyJS 插件不能编译版本超过 ES2015 (即ES6) 的代码. 这意味着如果你的代码使用了类、箭头函数或者语言的其它特性，你不能将它们编译成 ES5 版本的代码, 插件将抛出一个错误.  
  
> 如果你需要编译包含新的语法（的代码），使用[uglifyjs-webpack-plugin](https://github.com/webpack-contrib/uglifyjs-webpack-plugin) 包. This is the same plugin that’s bundled with webpack, but newer, and it’s able to compile the ES2015+ code.

### Loader-specific 选项

缩减代码的第二种方法是 loader-specific 选项 ([加载器是什么](https://webpack.js.org/concepts/loaders/)). 利用加载器选项，你可以压缩 minifier 不能缩减的东西。例如，当你利用[`css-loader`](https://github.com/webpack-contrib/css-loader)导入一个CSS文件时，该文件会被编译成一个字符串：

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

The minifier 不能压缩该代码，因为它是一个字符串。 为了缩减文件内容，我们需要像这样配置加载器：

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
* 其它流行的 minifiers: [Babel Minify](https://github.com/webpack-contrib/babel-minify-webpack-plugin), [Google Closure Compiler](https://github.com/roman01la/webpack-closure-compiler) 

## 指定 **NODE_ENV=production**

> ⭐️ **注意:** 这只适用于 webpack 3. 如果你在[production 模式下使用 webpack 4](#enable-the-production-mode),  `NODE_ENV=production` 优化已启用 – 自在地跳过该部分.

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

生产中通常不需要这些检查和警告，但是它们还是存在于代码中并增加了库的大小。 **在 webpack 4,** 通过添加`optimization.nodeEnv: 'production'` 选项以移除它们:

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    nodeEnv: 'production',
    minimize: true,
  },
}; 
```

**在 webpack 3,** 使用 [`DefinePlugin`](https://webpack.js.org/plugins/define-plugin/) 替代:

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

`optimization.nodeEnv` 选项和 `DefinePlugin` 工作方式相同 – 它们取代所有they replace all occurrences of `process.env.NODE_ENV` with the specified value. 使用上面的配置：

1.  Webpack will replace all occurrences of `process.env.NODE_ENV` with `"production"`:
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
2.  然后 [minifier](#enable-minification) 将会移除所有这些 `if` 分支 – 因为 `"production" !== 'production'` 总是错误的，插件明白这些分支中的代码永远不会执行：
    
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

当你使用 ES 模块, webpack 就可以进行 tree-shaking. Tree-shaking 是当 bundler 遍历整个依赖树时，检查使用了什么依赖，并移除没有用到的。所以，如果你使用了 ES 模块语法， webpack 可以去掉未使用的代码：

1.  你写了一个带有多个导出的文件，但是应用只使用它们其中的一个：
    
    ``` js
    // comments.js
    export const render = () => { return 'Rendered!'; };
    export const commentRestEndpoint = '/rest/comments';
    
    // index.js
    import { render } from './comments.js';
    render();
    ```
2.  Webpack 明白 `commentRestEndpoint` 没有用到并且不会在bundle中生成单独的导出点： 
    
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
  
> 然而，你不需要特定使用 webpack 内置的 minifier (`UglifyJsPlugin`). 任意的支持移除无用代码的 minifier (例如 [Babel Minify plugin](https://github.com/webpack-contrib/babel-minify-webpack-plugin) 或 [Google Closure Compiler plugin](https://github.com/roman01la/webpack-closure-compiler)) 都可以奏效.

> ❗ **警告:** 不要意外地将将 ES 模块编译为 CommonJS 模块。
  
> 如果你使用 Babel 的 `babel-preset-env` 或 `babel-preset-es2015`， 检查它们预先的设置。默认情况下， 它们将 ES 的 `import` 和 `export` 转译为 CommonJS 的 `require` 和 `module.exports`. [通过 `{ modules: false }` 选项](https://github.com/babel/babel/tree/master/packages/babel-preset-env) 来禁用它.  
  
> 与 TypeScript 相同：记得在你的 `tsconfig.json` 中设置 `{ "compilerOptions": { "module": "es2015" } }`。

### 扩展阅读

* [“深入 ES6 模块”](https://ponyfoo.com/articles/es6-modules-in-depth) 
* Webpack 文档 [关于 tree shaking](https://webpack.js.org/guides/tree-shaking/)  

## 优化图片

图片占页面大小的 [一半以上](http://httparchive.org/interesting.php?a=All&l=Oct%2016%202017)。 While they are not as critical as JavaScript (e.g., they don’t block rendering), they still eat a large part of the bandwidth. Use `url-loader`, `svg-url-loader` and `image-webpack-loader` to optimize them in webpack.

[`url-loader`](https://github.com/webpack-contrib/url-loader) inlines small static files into the app. Without configuration, it takes a passed file, puts it next to the compiled bundle and returns an url of that file. However, if we specify the `limit` option, it will encode files smaller than this limit as [a Base64 data url](https://css-tricks.com/data-uris/) and return this url. This inlines the image into the JavaScript code and saves an HTTP request:

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

> ⭐️ **Note:** Inlined images reduce the number of separate requests, which is good ([even with HTTP/2](https://blog.octo.com/en/http2-arrives-but-sprite-sets-aint-no-dead/)), but increase the download/parse time of your bundle and memory consumption. Make sure to not embed large images or a lot of them – or increased bundle time would outweigh the benefit of inlining.

[`svg-url-loader`](https://github.com/bhovhannes/svg-url-loader) works just like `url-loader` – except that it encodes files with the [URL encoding](https://developer.mozilla.org/en-US/docs/Glossary/percent-encoding) instead of the Base64 one. This is useful for SVG images – because SVG files are just a plain text, this encoding is more size-effective:

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

> ⭐️ **Note:** svg-url-loader has options that improve Internet Explorer support, but worsen inlining for other browsers. If you need to support this browser, [apply the `iesafe: true` option](https://github.com/bhovhannes/svg-url-loader#iesafe).

[`image-webpack-loader`](https://github.com/tcoopman/image-webpack-loader) compresses images that go through it. It supports JPG, PNG, GIF and SVG images, so we’re going to use it for all these types.

This loader doesn’t embed images into the app, so it must work in pair with `url-loader` and `svg-url-loader`. To avoid copy-pasting it into both rules (one for JPG/PNG/GIF images, and another one for SVG ones), we’ll include this loader as a separate rule with [`enforce: 'pre'`](https://webpack.js.org/configuration/module/#rule-enforce):

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

The default settings of the loader are already good to go – but if you want to configure it further, see [the plugin options](https://github.com/tcoopman/image-webpack-loader#options). To choose what options to specify, check out Addy Osmani’s excellent [guide on image optimization](https://images.guide/).

### Further reading

* ["What is base64 encoding used for?"](https://stackoverflow.com/questions/201479/what-is-base-64-encoding-used-for)   
* Addy Osmani’s [guide on image optimization](https://images.guide/)
    
## Optimize dependencies

More than a half of average JavaScript size comes from dependencies, and a part of that size might be just unnecessary.

For example, Lodash (as of v4.17.4) adds 72 KB of minified code to the bundle. But if you use only, like, 20 of its methods, then approximately 65 KB of minified code does just nothing.

Another example is Moment.js. Its 2.19.1 version takes 223 KB of minified code, which is huge – the average size of JavaScript on a page [was 452 KB in October 2017](http://httparchive.org/interesting.php?a=All&l=Oct%2016%202017). However, 170 KB of that size is [localization files](https://github.com/moment/moment/tree/4caa268356434f3ae9b5041985d62a0e8c246c78/locale). If you don’t use Moment.js with multiple languages, these files will bloat the bundle without a purpose.

All these dependencies can be easily optimized. We’ve collected optimization approaches in a GitHub repo – [check it out](https://github.com/GoogleChromeLabs/webpack-libs-optimizations)!

## Enable module concatenation for ES modules (aka scope hoisting)

> ⭐️ **Note:** if you’re using [webpack 4 with the production mode](#enable-the-production-mode), module concatenation is already enabled. Feel free to skip this section.

When you are building a bundle, webpack is wrapping each module into a function:

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

In the past, this was required to isolate CommonJS/AMD modules from each other. However, this added a size and performance overhead for each module.

Webpack 2 introduced support for ES modules which, unlike CommonJS and AMD modules, can be bundled without wrapping each with a function. And webpack 3 made such bundling possible – with [module concatenation](https://webpack.js.org/plugins/module-concatenation-plugin/). Here’s what module concatenation does:

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

See the difference? In the plain bundle, module 0 was requiring `render` from module 1. With module concatenation, `require` is simply replaced with required function, and module 1 is removed. The bundle has fewer modules – and less module overhead!

To turn on this behavior, **in webpack 4**, enable the `optimization.concatenateModules` option:

``` js
// webpack.config.js (for webpack 4)
module.exports = {
  optimization: {
    concatenateModules: true,
  },
};
```

**In webpack 3,** use the `ModuleConcatenationPlugin`:

``` js
// webpack.config.js (for webpack 3)
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin(),
  ],
};
```

> ⭐️ **Note:** Wonder why this behavior is not enabled by default? Concatenating modules is cool, [but it comes with increased build time and breaks hot module replacement](https://twitter.com/TheLarkInn/status/925800563144454144). That’s why it should only be enabled in production.

### Further reading

* Webpack docs [for the ModuleConcatenationPlugin](https://webpack.js.org/plugins/module-concatenation-plugin/)   
* [“Brief introduction to scope hoisting”](https://medium.com/webpack/brief-introduction-to-scope-hoisting-in-webpack-8435084c171f)  
* Detailed description of [what this plugin does](https://medium.com/webpack/webpack-freelancing-log-book-week-5-7-4764be3266f5)   

## Use `externals` if you have both webpack and non-webpack code

You might have a large project where some code is compiled with webpack, and some code is not. Like a video hosting site, where the player widget might be built with webpack, and the surrounding page might be not:

![A screenshot of a video hosting site](https://developers.google.com/web/fundamentals/performance/webpack/video-hosting.png)

(A completely random video hosting site)

If both pieces of code have common dependencies, you can share them to avoid downloading their code multiple times. This is done with [the webpack’s `externals` option](https://webpack.js.org/configuration/externals/) – it replaces modules with variables or other external imports.

### If dependencies are available in `window`

If your non-webpack code relies on dependencies that are available as variables in `window`, alias dependency names to variable names:

``` js
// webpack.config.js
module.exports = {
  externals: {
    'react': 'React',
    'react-dom': 'ReactDOM',
  },
};
```

With this config, webpack won’t bundle `react` and `react-dom` packages. Instead, they will be replaced with something like this:

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

### If dependencies are loaded as AMD packages

If your non-webpack code doesn’t expose dependencies into `window`, things are more complicated. However, you can still avoid loading the same code twice if the non-webpack code consumes these dependencies as [AMD packages](http://requirejs.org/docs/whyamd.html#amd).

To do this, compile the webpack code as an AMD bundle and alias modules to library URLs:

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

Webpack will wrap the bundle into `define()` and make it depend on these URLs:

``` js
// bundle.js (beginning)
define(["/libraries/react.min.js", "/libraries/react-dom.min.js"], function () { … });
```

If non-webpack code uses the same URLs to load its dependencies, then these files will be loaded only once – additional requests will use the loader cache.

> ⭐️ **Note:** Webpack replaces only those imports that exactly match keys of the `externals` object. This means that if you write `import React from 'react/umd/react.production.min.js'`, this library won’t be excluded from the bundle. This is reasonable – webpack doesn’t know if `import 'react'` and `import 'react/umd/react.production.min.js'` are the same things – so stay careful.

### Further reading

* Webpack docs [on `externals`](https://webpack.js.org/configuration/externals/)

## Summing up

* Enable the production mode if you use webpack 4
* Minimize your code with the bundle-level minifier and loader options
* Remove the development-only code by replacing `NODE_ENV` with `production`
* Use ES modules to enable tree shaking
* Compress images
* Apply dependency-specific optimizations
* Enable module concatenation
* Use `externals` if this makes sense for you
