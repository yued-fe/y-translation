
# 结论

> - **原文地址：** https://developers.google.com/web/fundamentals/performance/webpack/conclusion
> - **原文作者：** [Ivan Akulov](https://developers.google.com/web/resources/contributors/iamakulov)
> - **译文地址：** https://github.com/yued-fe/y-translation/blob/master/en/Web-Performance-Optimization-with-webpack/Conclusion.md
> - **译者：** [闫萌](https://github.com/yanyixin)
> - **校对者：**

总结:

* **减少不必要的字节。** 压缩所有资源，去除无用代码，谨慎添加依赖。
* **通过路由分离代码。** 只加载当前真正需要的资源，其他的资源延迟加载。
* **缓存代码。** 应用中某部分代码的更新频率比其他部分低。可以将这部分代码剥离，以便只在必要时重新加载。
* **持续关注应用大小。** 使用像 [webpack-dashboard](https://github.com/FormidableLabs/webpack-dashboard/) 和 [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) 这类的工具来了解应用的大小。然后每隔几个月重新检查一下应用的性能。

Webpack 不是唯一一个可以帮助你让应用变得更快的工具。还可以考虑让你的应用变成一个渐进式 web 应用（Progressive Web App），从而获得更好的体验，并且使用像 [Lighthouse](https://developers.google.com/web/tools/lighthouse/) 这样的自动分析工具来获得改进意见。

别忘了去阅读 [webpack 文档](https://webpack.js.org/guides/) - 那里还有大量有用的信息。

同时记得用 [training 项目](https://github.com/GoogleChromeLabs/webpack-training-project)来练习哦！
