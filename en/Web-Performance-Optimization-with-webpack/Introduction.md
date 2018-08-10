
# 介绍

> - **原文地址：** https://developers.google.com/web/fundamentals/performance/webpack/
> - **原文作者：** [Addy Osmani](https://developers.google.com/web/resources/contributors/addyosmani)
> - **译文地址：** https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Introduction.md
> - **译者：** [闫萌](https://github.com/yanyixin)
> - **校对者：**

现代 web 应用经常使用**打包工具**来创建生产环境的“打包”文件（脚本、样式等等），这些文件经过[优化](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/javascript-startup-optimization)和[压缩](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer)之后能够极快的被用户下载。在**使用 webpack 进行 web 性能优化**系列文章中，我们将介绍如何使用 webpack 高效的优化站点资源。这将会帮助用户更快的加载网站从而进行交互。

![webpack logo](https://developers.google.com/web/fundamentals/performance/webpack/webpack-logo.png)

webpack 是当下最流行的打包工具之一。我们可以利用其特性来优化代码，通过[代码拆分](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching#lazy-loading)可以将脚本拆分为核心和非核心部分，并且去除无用的代码（这仅仅是一小部分的优化案例），从而确保你的应用具有最小的网络负担和处理成本。

![Before and after applying JavaScript
  optimizations. Time-to-Interactive is improved](https://developers.google.com/web/fundamentals/performance/webpack/code-splitting.png)

受 Susie Lu 的[在 Bundle Buddy 中进行代码拆分](http://www.susielu.com/data-viz/bundle-buddy)的启发

> **⭐️ 注意：** 我们创建了一个可供练习的应用来演示这篇文章中讲到的优化内容。请尽可能抽出更多的时间来练习这些技巧：[`webpack-training-project`](https://github.com/GoogleChromeLabs/webpack-training-project)

让我们从现今应用中最耗费资源之一的 JavaScript 开始优化。

* [Decrease Front-end Size](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size)
* [利用好持久化缓存](https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Make-Use-of-Long-term-Caching.md)
* [Monitor and analyze the app](https://developers.google.com/web/fundamentals/performance/webpack/monitor-and-analyze)
* [Conclusions](https://developers.google.com/web/fundamentals/performance/webpack/conclusion)
