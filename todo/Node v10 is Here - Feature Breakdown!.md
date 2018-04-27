* 原文地址：[node-js-10-lts-feature-breakdown](https://blog.risingstack.com/node-js-10-lts-feature-breakdown/)
* 译者：[周文康](https://github.com/wenkangzhou)
* 校对者：

# Node V10 的特性解析

Node.js 遵循一个发布计划，即每半年从主分支中抽离出一个主要版本。每年十月，新的奇数版本中断，最新的偶数版本转换到LTS计划。每年的4月都会标记新的偶数编号版本的日期，今天正好是 Node V10 从主分支抽离出来的日子。
![](https://blog.risingstack.com/content/images/2018/04/whats-new-in-node-js-10-risingstack.png)

我们来看看 Node 10 带来的新功能！

## Node 10 中的 HTTP/2 更稳定

[HTTP/2](https://en.wikipedia.org/wiki/HTTP/2)的支持在 Node v8.4.0 中作为实验功能出现。在 Node v10 中，http2 模块已经成为 Node 核心的稳定补充。你可以自己使用它 - 请参阅我们发表的文章
[Node.js & HTTP/2 Push here](https://blog.risingstack.com/node-js-http-2-push/)。

HTTP/2 是一种支持 TCP 多路复用的二进制协议，这意味着 TCP 握手只能处理一次，服务器可以重用已有的连接，通过同一个连接发送多个请求的响应。它还支持服务器推送，所以当浏览器请求一个 HTML 文件时，你可以在页面加载和解析之前发送必要的 JavaScript 脚本和 CSS 样式表。浏览器意识到它需要更多的回路去请求所有必要的信息来正确呈现网站。

但是，有一个问题，因为浏览器只支持 SSL 的 HTTP/2 。SSL 终端是一项 CPU 密集型任务，因此它应该由 edge 代理处理。 Nginx 从1.13.9版本以后支持 HTTP/2 提供的所有功能。在你的 Node 服务器里有一个 edge 代理是最好的做法，如果你不希望你的 Node 进程以超级用户身份运行，你可以公开80和443端口，并且这样你可以把 HTTP 推送和 TLS 终端设置在你的服务器代码之外。

如果你在合适安全的 DMZ（两个防火墙之间的空间）里使用处在微服务环境的 HTTP/2，你就可以直接暴露你的 Node 服务器 - 这样你就可以享受 HTTP/2 的所有优势，而不会因 Node 进程引起任何麻烦。

截至目前，你可以使用带有 hapi 和 koa 的 HTTP2 模块。根据这个[github issue](https://github.com/expressjs/express/issues/2364)，该 `spdy` 模块完全支持 HTTP/2，所以你可以使用 express ，或者你如果觉得不保险，可以使用[`express-http2-workaround`](https://www.npmjs.com/package/express-http2-workaround)模块。

你会发现大多数例子会告诉你，你应该使用 `http2.createSecureServer` 但正如我们上面所讨论的，你应该让你的 edge 代理处理它。

### HTTP/2 and express

```
const express = require('express')
const spdy = require('spdy')

const app = express()

app.get('/', function (req, res) {
 res.send('Hello, World!')
})

spdy.createServer(options, app).listen(3000, err => {
 if (err) {
   throw new Error(err)
 }

 console.log('Listening on port: 3000.')
})

```

### HTTP/2 and koa

```
const http2 = require('http2');
const koa = require('koa');
const router = require('koa-route');
const fs = require('fs');

const app = koa();

app.use(router.get('/', function *(next) {
 this.body = 'Hello, World!';
 yield next;
}));

http2
 .createServer({}, app.callback())
 .listen(3000, (err) => {
   if (err) {
     throw new Error(err);
   }

   console.log('Listening on port: 3000.');
});

```

### HTTP/2 and Hapi

```
const fs = require('fs')
const Hapi = require('hapi')
const http2 = require('http2')

const server = new Hapi.Server()
server.connection({
 listener: http2.createServer(),
 host: 'localhost',
 port: 3000
})

server.route({
 method: 'GET',
 path:'/hello',
 handler: function (request, reply) {
   return reply('Hello, World!')
 }
})

server.start((err) => {
 if (err) console.error(err)
 console.log(`Started ${server.connections.length} connections`)
})

```

## Node.js v10中的ESM模块

虽然浏览器受到这样一个事实的困扰，即使您可以从 HTML 加载多个脚本文件，但这些文件不能被视为模块，因为所有对象都位于全局名称空间中。你可以使用[模块模式](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)工作，但它仍然不是很方便。随着捆绑器和 ECMAScript 模块（简称 ES 模块或 ESM ）的出现，这个问题有所缓解，但尚未完全解决。

另一方面，Node.js - 作为一个服务器框架 - 从一开始就带有一个名为 CommonJS 的模块系统。

问题在于这两个模块系统不兼容，所以[模块团队](https://github.com/nodejs/modules)必须找到合适的解决方案，以便 JavaScript 模块可以在 Node 和浏览器中跨平台使用。

Node.js v10 不能完全实现 ESM，但我们一定会看到有关该主题的快速迭代。欲了解更多关于 ES 模块的信息，请阅读 Axel Rauschmayer 博士的 [excellent post](http://2ality.com/2014/09/es6-modules-final.html).

## Error Codes

在 Node v10 之前，匹配 `catch` 子句中的错误的唯一方法是检查错误消息。

```
try {
 // do stuff
} catch (err) {
 if (err.message === 'Expected error message') {
   // handle specific error
 } else {
   // general error handler
 }
}

```


这种方法的问题是用户在处理错误时只能匹配错误消息。如果错误信息中存在拼写错误，或者您认为它不具有足够的描述性，则需要使用主要版本缓冲区，因为其他错误信息取决于错误消息中的确切字符串。

在 Node.js v10 中，Node.js API 抛出的所有错误都有错误代码，这意味着您不需要匹配错误信息像以前那样具有很强的可读性。

因此，当您迁移到 Node v10 时，您可以将代码更改为以下内容：

```
try {
 // do stuff
} catch (err) {
 if (err.code === 'ERR_ERROR_CODE') {
   // handle specific error
 } else {
   // general error handler
 }
}

```


要找出您需要使用哪个特定的错误代码，请检查[文档](https://nodejs.org/dist/latest/docs/api/errors.html#errors_node_js_error_codes)

你可以在[这里](https://medium.com/the-node-js-collection/node-js-errors-changes-you-need-to-know-about-dc8c82417f65)阅读更多关于错误代码
## 实验性功能-Fs promises

Node.js v8 开始引入了`util.promisify`以便轻松包装提供回调API的函数。在 `fs` 的最新版本中，消除了旧方式的额外步骤和开销。

## Node 10中的N-API

原生模块一直是 Node.js 中的一个难点，特别是在切换版本时。在 Node v8 之前，本地模块必须直接依赖 V8/[NAN](https://github.com/nodejs/nan) Apis。这导致 API/ABI 稳定性缺乏保证，并迫使本地插件开发人员更新或至少在每个主要版本中重新编译其代码。

N-API 提供了 V8/NAN API 上的抽象层，因此可以在更高级别的层中处理这些变化，从而使本地插件的开发人员在表现层使用起来更稳定。到目前为止，这是一个实验性功能，但在 Node v10 中它已被提升到稳定状态，因此如果您尚未这样做，可能是时候开始尝试并可能迁移到它了。

N-API 也是 VM 多样性的主要垫脚石。Node.js 最初只运行在 Chrome 的 V8 上，但最近它在微软的 [ChakraCore](https://github.com/Microsoft/ChakraCore) 上的实现正在开发中。通过使用 N-API，为不同的虚拟机创建绑定更加容易，因此将 Node.j移植到其他运行时会比现在更容易。它对于物联网开发人员尤其有用，所以很快您可能实际上能够在您的冰箱上运行 Node.js。

## V8 6.6：向异步生成器和数据性能改进问好

Node.js 随附 V8 v6.6，带来异步生成器和数据性能改进。`Array.reduce` 把多重数组的速度提高了10倍。异步生成器和异步迭代的性能也大幅提高。

新版本也提供了新的 JavaScript 语言功能。完整列表可以在[这里](https://v8project.blogspot.hu/)找到

## Node.js的下一步是什么？

有一件事是可以肯定的，Node 会不断改进，并且开发背后的优秀人才会添加更多的功能。如果您想跟踪 Node 11.0.0会发生什么事情 - 我建议关注[这里](https://github.com/nodejs/Release/issues/328)。
