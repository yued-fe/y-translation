Node.js遵循一个发布计划，即每半年从主分支中抽离出一个主要版本。每年十月，新的奇数版本中断，最新的偶数版本转换到LTS计划。每年的4月都会标记新的偶数编号版本的日期，今天正好是Node V10从主分支抽离出来的日子。
![](https://blog.risingstack.com/content/images/2018/04/whats-new-in-node-js-10-risingstack.png)

我们来看看Node 10带来的新功能！

## Node 10中的HTTP/2更稳定

[HTTP/2](https://en.wikipedia.org/wiki/HTTP/2)的支持在Node v8.4.0中作为实验功能出现。使用Node v10，http2模块已经成为Node核心的稳定补充。你可以自己使用它 - 请参阅我们在
[Node.js & HTTP/2 Push here](https://blog.risingstack.com/node-js-http-2-push/)。

HTTP/2是一种支持TCP多路复用的二进制协议，这意味着TCP握手只能处理一次，服务器可以重用已有的连接，通过同一个连接发送多个请求的响应。它还支持服务器推送，所以当浏览器请求一个HTML文件时，你可以在页面加载和解析之前发送必要的JavaScript脚本和CSS样式表。浏览器意识到它需要更多的回路去请求所有必要的信息来正确呈现网站。

但是，有一个问题，因为浏览器只支持SSL的HTTP/2。SSL终端是一项CPU密集型任务，因此它应该由edge代理处理。Nginx从1.13.9版本以后支持HTTP/2提供的所有功能。在您的Node服务器里有一个edge代理是最好的做法，如果您不希望你的Node进程以超级用户身份运行，您可以公开80和443端口，并且这样您可以把HTTP推送和TLS终端设置在你的服务器代码之外。

假如你在一个微服务环境，切处于合适安全的DMZ（两个防火墙之间的空间）中使用HTTP/2，你就可以直接暴露你的Node服务器 - 这样您就可以享受HTTP/ 2的所有优势，而不会因Node进程引起任何麻烦。

截至目前，您可以使用带有hapi和koa的HTTP2模块。根据这个[github issue](https://github.com/expressjs/express/issues/2364)，该`spdy`模块具有完整的HTTP/2支持，所以你可以使用express，或者你如果觉得不保险，可以使用[`express-http2-workaround`](https://www.npmjs.com/package/express-http2-workaround)模块。

你会发现大多数例子会告诉你，你应该使用`http2.createSecureServer`但正如我们上面所讨论的，你应该让你的边缘代理处理它。

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

虽然浏览器受到这样一个事实的困扰，即使您可以从HTML加载多个脚本文件，但这些文件不能被视为模块，因为所有对象都位于全局名称空间中。你可以使用[模块模式](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)工作，但它仍然不是很方便。随着捆绑器和ECMAScript模块（简称ES模块或ESM）的出现，这个问题有所缓解，但尚未完全解决。

另一方面，Node.js - 作为一个服务器框架 - 从一开始就带有一个名为CommonJS的模块系统。

问题在于这两个模块系统不兼容，所以[模块团队](https://github.com/nodejs/modules)必须找到合适的解决方案，以便JavaScript模块可以在Node和浏览器中跨平台使用。

Node.js v10不能完全实现ESM，但我们一定会看到有关该主题的快速迭代。欲了解更多关于ES模块的信息，请阅读Axel Rauschmayer博士的[[excellent post]](http://2ality.com/2014/09/es6-modules-final.html).

## Error Codes

在Node v10之前，匹配`catch`子句中的错误的唯一方法是检查错误消息。

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

在Node.js v10中，Node.js API抛出的所有错误都有错误代码，这意味着您不需要匹配错误信息像以前那样具有很强的可读性。

因此，当您迁移到Node v10时，您可以将代码更改为以下内容：

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

Node.js v8开始引入了`util.promisify`以便轻松包装提供回调API的函数。在`fs`的最新版本中，消除了旧方式的额外步骤和开销。

## Node 10中的N-API

原生模块一直是Node.js中的一个难点，特别是在切换版本时。在Node v8之前，本地模块必须直接依赖V8 / [NAN](https://github.com/nodejs/nan) Apis。这导致API / ABI稳定性缺乏保证，并迫使本地插件开发人员更新或至少在每个主要版本中重新编译其代码。

N-API提供了V8 / NAN API上的抽象层，因此可以在更高级别的层中处理这些变化，从而使本地插件的开发人员在表现层使用起来更稳定。到目前为止，这是一个实验性功能，但在Node v10中它已被提升到稳定状态，因此如果您尚未这样做，可能是时候开始尝试并可能迁移到它了。

N-API也是VM多样性的主要垫脚石。Node.js最初只运行在Chrome的V8上，但最近它在微软的[ChakraCore](https://github.com/Microsoft/ChakraCore)上的实现正在开发中。通过使用N-API，为不同的虚拟机创建绑定更加容易，因此将Node.js移植到其他运行时会比现在更容易。它对于物联网开发人员尤其有用，所以很快您可能实际上能够在您的冰箱上运行Node.js。

## V8 6.6：向异步生成器和数据性能改进问好

Node.js随附V8 v6.6，带来异步生成器和数据性能改进。`Array.reduce`把多重数组的速度提高了10倍。异步生成器和异步迭代的性能也大幅提高。

新版本也提供了新的JavaScript语言功能。完整列表可以在[这里](https://v8project.blogspot.hu/)找到

## Node.js的下一步是什么？

有一件事是可以肯定的，Node会不断改进，并且开发背后的优秀人才会添加更多的功能。如果您想跟踪Node 11.0.0会发生什么事情 - 我建议关注[这里](https://github.com/nodejs/Release/issues/328)。