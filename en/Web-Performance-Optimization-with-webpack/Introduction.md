
# Introduction

> - **原文地址：** https://developers.google.com/web/fundamentals/performance/webpack/
> - **原文作者：** [Addy Osmani](https://developers.google.com/web/resources/contributors/addyosmani)
> - **译文地址：** https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Introduction.md
> - **译者：** 
> - **校对者：**

Modern web applications often use a **bundling tool** to create a production "bundle" of files (scripts, stylesheets, etc.) that is [optimized](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/javascript-startup-optimization), [minified](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/optimize-encoding-and-transfer) and can be downloaded in less time by your users. In **Web Performance Optimization with webpack**, we will walk through how to effectively optimize site resources using [webpack](https://webpack.js.org/). This can help users load and interact with your sites more quickly.

![webpack logo](https://developers.google.com/web/fundamentals/performance/webpack/webpack-logo.png)

webpack is one of the most popular bundling tools in use today. Taking advantage of its features for optimizing modern code, [code-splitting](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching#lazy-loading) scripts into critical and non-critical pieces and stripping out unused code (to name but a few optimizations) can ensure your app has a minimal network and processing cost.

![Before and after applying JavaScript
  optimizations. Time-to-Interactive is improved](https://developers.google.com/web/fundamentals/performance/webpack/code-splitting.png)  

Inspired by [Code-splitting in Bundle Buddy](http://www.susielu.com/data-viz/bundle-buddy) by Susie Lu

> **⭐️ Note:** We created a training app to play with optimizations described in this article. Try squeezing the most out of it to practice the tips: [`webpack-training-project`](https://github.com/GoogleChromeLabs/webpack-training-project)

Let’s get started by looking at optimizing one of the costliest resources in a modern app – JavaScript.

* [Decrease Front-end Size](https://developers.google.com/web/fundamentals/performance/webpack/decrease-frontend-size)
* [Make Use of Long-term Caching](https://developers.google.com/web/fundamentals/performance/webpack/use-long-term-caching)
* [Monitor and analyze the app](https://developers.google.com/web/fundamentals/performance/webpack/monitor-and-analyze)
* [Conclusions](https://developers.google.com/web/fundamentals/performance/webpack/conclusion)
