<div class="kg-card-markdown">

Node.js follows a release plan where every major release is cut from the master branch in every 6 months. In every October, new odd-numbered versions are cut and the latest even-numbered release transitions to the LTS plan. Each year’s April marks the date of the new even numbered release and today is the day when Node v10 is cut from the master.

![whats-new-in-node-js-10-risingstack](https://blog.risingstack.com/content/images/2018/04/whats-new-in-node-js-10-risingstack.png)

Let’s take a look at the new features Node 10 brings!

## Stable HTTP/2 in Node 10

The support for [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) has landed in Node v8.4.0 as an experimental feature. With Node v10 the http2 module has become a stable addition to the Node core. You can use it on its own - see our post on [Node.js & HTTP/2 Push here](https://blog.risingstack.com/node-js-http-2-push/).

HTTP/2 is a binary protocol that supports TCP multiplexing, which means that TCP handshakes have to be handled only once and the server can reuse an already existing connection to send the response of multiple requests through the same connection. It also supports server push, so when the browser requests an HTML file, you can send along the necessary JavaScript scripts and CSS stylesheets before the page is loaded and parsed. The browser realizes that it will need more round trips to requests all the necessary information to properly render the site.

There’s a catch, however, as browsers only support HTTP/2 over SSL. SSL termination is a CPU intensive task so it should be handled by the edge proxy. Nginx supports all the functionality HTTP/2 provides since its [1.13.9](https://www.nginx.com/blog/nginx-1-13-9-http2-server-push/) release. It is a best practice to have an edge proxy in front of your Node server, so you can expose port 80 and 443 as you don’t want your Node process to run as superuser and this way you can setup HTTP push and TLS termination outside your server code.

If you use HTTP/2 in a microservices environment in a properly secured DMZ, you can expose your Node server directly - so you can enjoy all the benefits of HTTP/2 without any hassle straight from your Node process.

As of right now, you can use the HTTP2 module with hapi and koa out of the box. According to this [github issue](https://github.com/expressjs/express/issues/2364) the `spdy` module has full HTTP/2 support so you can use that with express, or you can use the [`express-http2-workaround`](https://www.npmjs.com/package/express-http2-workaround) module if you feel adventurous.

Most of the examples you’ll find will tell you that you should use `http2.createSecureServer` but as we discussed above, you should let your edge proxy handle that.

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

## ESM Modules in Node.js v10

While browsers were plagued by the fact that even though you can load multiple script files from HTML, those could not be considered as modules because all your objects lived in the global namespace. You could work this around using the [module pattern](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript), but it still wasn’t really convenient. With the advent of bundlers and ECMAScript Modules (ES Modules or ESM for short) this problem is somewhat mitigated, but not yet fully solved.

On the other hand, Node.js - being a server framework - came with a module system called CommonJS baked in from the very beginning.

The problem is that the two module systems are not compatible so the [Modules Team](https://github.com/nodejs/modules) had to find a proper solution so JavaScript modules could be built in a platform agnostic way and could be used both in Node and in browsers.

Node.js v10 does not bring the full implementation of ESMs, but we will definitely see rapid iterations regarding the topic. To learn more about ES Modules read Dr. Axel Rauschmayer’s [excellent post](http://2ality.com/2014/09/es6-modules-final.html).

## Error Codes

Before Node v10 the only way to match errors in `catch` clauses was to check for the error message.

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

The problem with this approach is that users can only match for the error message when handling errors. If there’s a typo in the error message, or you decide that it’s not descriptive enough, you need a major version bump as others depend on the exact string in your error message.

In Node.js v10 all errors thrown by the Node.js APIs have an error code as well, meaning you don’t need to match the error message that should be readable for humans to begin with.

So you can change your code to the following when you migrate to Node v10:

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

To find out which specific error code you need to use, check the [documentation](https://nodejs.org/dist/latest/docs/api/errors.html#errors_node_js_error_codes)

You can read more about error codes [here](https://medium.com/the-node-js-collection/node-js-errors-changes-you-need-to-know-about-dc8c82417f65)

## Experimental Fs promises

Node.js v8 introduced `util.promisify` to easily wrap functions that provide a callback API. In the latest release functions of the `fs` return promises directly, eliminating the extra step and overhead of the old way.

## N-API in Node 10

Native modules have always been a pain point in Node.js, especially when switching versions. Before Node v8, native modules had to directly depend on the V8 / [NAN](https://github.com/nodejs/nan) Apis. That caused a lack of API / ABI stability guarantees and forced native addon developers to update or at least recompile their code with every major release.

The N-API provides an abstraction layer over the V8 / NAN APIs so changes in those can be handled in a higher level layer, resulting in a more stable surface that native addon developers can use. So far it was an experimental feature, but it has been promoted to stable in Node v10, so it might be time to start experimenting and maybe migrating to it if you haven’t already done so.

The N-API is also a major stepping stone towards VM-diversity. Node.js originally ran only on Chrome’s V8, but lately it’s implementation on Microsoft’s [ChakraCore](https://github.com/Microsoft/ChakraCore) is under development. By using the N-API, it is easier to create bindings for different VMs, so porting Node.js to other runtimes will be a lot easier than it is now. It will especially be useful for IoT developers, so soon you might be actually able to run Node.js on your fridge.

## V8 6.6: Say hello to async generators & array performance improvements

Node.js is shipped with the V8 v6.6 that brings async generators and array performance improvements. `Array.reduce` has become 10 times faster for holey double arrays. The performance of async generators and async iteration has also been increased by a great margin.

The new release provides new JavaScript language features as well. The full list can be found [here](https://v8project.blogspot.hu/)

## What’s next for Node.js?

One thing is for sure, Node will keep on improving and the fine people behind it’s development are going to add even more features. If you’d like to keep track on what’s going to happen with Node 11.0.0 - I recommend to follow [this thread](https://github.com/nodejs/Release/issues/328).

</div>
