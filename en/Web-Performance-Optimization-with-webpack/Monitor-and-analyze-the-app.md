# 监控和分析应用

> - **原文地址：** [monitor and analyze](https://developers.google.com/web/fundamentals/performance/webpack/monitor-and-analyze)
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** [监控和分析应用](https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Monitor-and-analyze-the-app.md)
> - **译者：** 泥坤
> - **校对者：** [杨建](https://github.com/ASkyBig)、[闫蒙](https://github.com/yanyixin)

即使你可以通过配置 webpack 使得应用尽可能小，追踪它并且知道它包含什么仍然是很重要的。否则，你可能安装了一个
让应用大了两倍的依赖却浑然不觉。

这节就来讲几个可以帮助你深入分析 bundle 的工具。

## 追踪 bundle 的大小

为了监控你的应用大小，可以在开发过程中使用 [webpack-dashboard](https://github.com/FormidableLabs/webpack-dashboard/) 和在 CI 上使用 [bundlesize](https://github.com/siddharthkp/bundlesize)。

### webpack-dashboard

[webpack-dashboard](https://github.com/FormidableLabs/webpack-dashboard/) 增强了 webpack 的输出，包含依赖的大小、进度和其他细节。这是它的界面：

![](https://developers.google.com/web/fundamentals/performance/webpack/webpack-dashboard.png)

这个看板帮助追踪大的依赖 —— 如果你增加了一个依赖，那么你就能够立刻在 Modules 部分中看到它。

要想使用它，需要安装依赖包 `webpack-dashboard` :

```js
npm install webpack-dashboard --save-dev
```

然后在 `webpack.config.js` 文件中的 `plugins` 字段里增加这个 plugin:

```js
// webpack.config.js
const DashboardPlugin = require('webpack-dashboard/plugin');

module.exports = {
  plugins: [
    new DashboardPlugin(),
  ],
};
```

如果你使用基于 express 的服务，也可以使用 `compiler.apply()`:

``` js
compiler.apply(new DashboardPlugin());
```

尽情地使用 dashboard 来找到可能优化的地方吧！举个例子，纵览 **Modules** 部分可以找到过大的库，然后用相对小的库来替代掉它。

### bundlesize

[bundlesize](https://github.com/siddharthkp/bundlesize) 校验 webpack 的资源有没有超过指定的大小。将它集成到 CI 中，就可以在应用过大的时候收到提醒。

![](https://developers.google.com/web/fundamentals/performance/webpack/bundlesize.jpg)

配置:

**确定大小上限**

1. 先优化应用，让它尽可能小，然后运行 prodocution build；
2. 在 `package.json` 中增加 `bundlesize ` 字段的配置，如下：

    ``` js
    // package.json
    {
      "bundlesize": [
        {
          "path": "./dist/*"
        }
      ]
    }
    ```

3.  用 [npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b) 执行 `bundlesize`:

    ``` js
    npx bundlesize
    ```

    将打印出每个文件 gzip 过后的大小
    
    ``` js
    PASS  ./dist/icon256.6168aaac8461862eab7a.png: 10.89KB
    PASS  ./dist/icon512.c3e073a4100bd0c28a86.png: 13.1KB
    PASS  ./dist/main.0c8b617dfc40c2827ae3.js: 16.28KB
    PASS  ./dist/vendor.ff9f7ea865884e6a84c8.js: 31.49KB
    ```

4. 在每个文件大小的基础上增加 10-20%, 就可以得到大小上限。这 10%-20% 的 buffer 可以保证既不妨碍你日常开发，又可以在它的体积增长过快的时候向你报警。

**启用 `bundlesize`**

5. 安装 <code>bunlesize</code> 包作为开发依赖:

    ``` js
    npm install bundlesize --save-dev
    ```

6. 在 `package.json` 的 `bundlesize` 字段里，指定具体的大小上限，对于有的文件（例如图片），你可能需要为特定的文件类型设置上限，而不是每个文件。

    ``` js
    // package.json
    {
      "bundlesize": [
        {
          "path": "./dist/*.png",
          "maxSize": "16 kB",
        },
        {
          "path": "./dist/main.*.js",
          "maxSize": "20 kB",
        },
        {
          "path": "./dist/vendor.*.js",
          "maxSize": "35 kB",
        }
      ]
    }
    ```

7. 增加 npm 脚本来运行检查。

    ``` js
    // package.json
    {
      "scripts": {
        "check-size": "bundlesize"
      }
    }
    ```

8.  配置好 CI ，以便在每次 push 的时候运行 <code>npm run check-size</code> （如果你在 github 上开发项目的话还可以[在 github 中集成 `bundlesize`](https://github.com/siddharthkp/bundlesize#2-build-status))。

 就是这样！现在如果你运行 `npm run check-size` 或者 push 代码，你会看到输出文件是否足够小。


![](https://developers.google.com/web/fundamentals/performance/webpack/bundlesize-output-success.png)

如果是失败的情况：

![](https://developers.google.com/web/fundamentals/performance/webpack/bundlesize-output-failure.png)

### 扩展阅读:

- Alex Russell [about the real-world loading time we should
target](https://infrequently.org/2017/10/can-you-afford-it-real-world-web-performance-budgets/)

## 分析 bundle 为什么这么大

你可能想要深入研究 bundle，来分析哪些模块占用了空间。请看 [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer):

<figure>
  <video src="https://developers.google.com/web/fundamentals/performance/webpack/webpack-bundle-analyzer.mp4" alt="A screen recording of the webpack bundle analyzer
page" autoplay controls loop></video>
  <figcaption>(Screen recording from <a
href="https://github.com/webpack-contrib/webpack-bundle-analyzer">github.com/webpack-contrib/webpack-bundle-analyzer</a>)
</figcaption>
</figure>

webpack-bundle-analyzer 扫描 bundle 并且建立一个可视化的结果展示它包含什么，通过该可视化界面可以去找大的或者不必要的依赖。

要使用 analyzer，需要安装 `webpack-bundle-analyzer`。

``` js
npm install webpack-bundle-analyzer --save-dev
```

在 webpack 配置文件里增加一个 plugin。

```js
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin(),
  ],
};
```
然后运行 production build，插件会在浏览器里打开一个统计的页面。

默认情况下，统计页面展示的是解析过后的文件大小。（即出现在 bundle 里的文件），你可能想要比较 gzip 压缩之后的大小，因为这会更接近真实用户的体验。可以在侧边栏切换大小类型。

> ⭐️注意 : 如果你使用 [ModuleConcatenationPlugin](https://webpack.js.org/plugins/module-concatenation-plugin/)，一部分在 webpack-bundle-analyzer 输出的模块可能会被合并，使得报告的信息量减少。所以如果你在用这个插件，在分析的过程中需要将它禁用掉。

以下是希望从报告中得到的信息：

- **大的依赖** 为什么他们那么大？是否有更小的替代品（例如 Preact 代替 React）？你是否需要其中的全部代码？（例如，Moment.js 包含很多并不常使用并可以扔掉的本地化语言环境 ）

- **重复的依赖** 你是否在多个文件里看到相同的依赖？（使用像 webpack4 中的 `optimization.splitChunks.chunks` 选项或者 `webpack3 中的 CommonsChunkPlugin` 来将他们移到一个公共文件里）或者 bundle 是否包含了同一个库的多个版本？

- **相似的依赖** 是否有相似的库做着差不多的事情？（例如 `comment` 和 `date-fns` 或者 `lodash` 和 `lodash-es`）试着统一成单一的工具。

同时，建议看一下 Sean Larkin 的 [great analysis of webpack
bundles](https://medium.com/webpack/webpack-bits-getting-the-most-out-of-the-commonschunkplugin-ab389e5f318)。

## 总结

- 使用 webpack-dashboard 和 bundlesize 来持续关注你的应用大小。

- 用 `webpack-bundle-analyzer` 来深究应用大小的构成
