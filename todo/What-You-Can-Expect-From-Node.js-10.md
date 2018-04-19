
- 原文地址：https://nodesource.com/blog/what-you-can-expect-from-node-js-10
- 译者：[名称](git、知乎、掘金、个人网站或者微博的链接)
- 校对者：[名称](git、知乎、掘金、个人网站或者微博的链接)

The release of Node.js v10 is just a week away, and will include a suite of new features including updates to errors and the formal addition of N-API.

Beyond these new additions, I believe we will continue to see increased adoption and development of Node.js features that have been recently proposed or otherwise landed since the release Node.js 8.

## Let's take a closer look at what we can expect from Node.js v10:Codes for Errors in Node.js 10

There's a rather large change coming to errors in Node.js: **errors will have codes that follow a consistent and repeatable scheme**.

Previously, any kind of change to errors needed to be shipped in a semver major release. This became a major pain point, and is exemplified by something as trivial as wanting to correct a typo in an error, but needing to wait until the next major version of Node.js to ship.

This has the added benefit of helping normalize errors across platforms, making for a more consistent development experience no matter the operating system being used.

If you’d like to learn more about the new implementation of error codes in Node.js, be sure to check out Michael Dawson's post about it [here](https://medium.com/the-node-js-collection/node-js-errors-changes-you-need-to-know-about-dc8c82417f65).

## N-API: No Longer Experimental

A [pull request](https://github.com/nodejs/node/pull/19262) landed last month to change the status of N-API from Experimental to Stable. While the addition of N-API won't visibly affect the majority of users and module developers, its goal is simple: remove barriers caused by native modules when upgrading Node.js.

Native modules have consistently been a necessary pain point, and aren't as easy to "fix" for the average developer. For more info, check out the [article on N-API](https://medium.com/the-node-js-collection/n-api-next-generation-node-js-apis-for-native-modules-169af5235b06) by Michael Dawson of IBM and Arunesh Chandra of Microsoft.

## HTTP/2 in Node.js 10

The implementation of HTTP/2 in Node.js landed in Node.js 8 LTS, [as of Node.js 8.4.0](https://github.com/nodejs/node/pull/14811). That said, there hasn’t been much discourse around or a major move towards adoption of the new implementation beyond the bounds of the Node.js core team.

The HTTP/2 implementation is a pretty great addition to Node.js core, and is,in my opinion, important for the continued evolution of Node.js for web applications and the web platform. For more context on the release and usage of HTTP/2, checkout James Snell's [post on the subject](https://medium.com/the-node-js-collection/say-hello-to-http-2-for-node-js-core-261ba493846e).

## ESM and Node.js

ECMAScript Modules (a.k.a. ES Modules or ESM) are one of the most challenging and important hurdles for Node.js in the coming future. ECMAScript, which is what we're actually using when we use "[JavaScript™](http://tsdr.uspto.gov/#caseNumber=75026640&caseType=SERIAL_NO&searchType=statusSearch)", outlined its own modules system ECMAScript 2015 (ES6) spec.

The new, native implementation is at odds with how Node.js has implemented modules. This has caused a significant amount of discourse, both within the Node.js project and the broader JavaScript community, who now largely use both Node.js and npm as a platform for ecosystem tooling and module delivery.

We’re not going to see a full implementation of ESM in Node.js v10, but we are seeing continuous iteration and development in this area. The [Modules Team](https://github.com/nodejs/modules) was spun up a few months ago, and has been actively working on discussing the needs and implementation details around Node.js and ESM. This team is one of the largest active teams in Node.js, with over 30 active contributors.

For more info on ES Modules and Node.js, check out Myles Borins [post on the subject](https://medium.com/the-node-js-collection/the-current-state-of-implementation-and-planning-for-esmodules-a4ecb2aac07a).

## Continued Adoption of Async Hooks

Async Hooks shipped as Experimental in Node.js 8, and has since gained some traction around the ecosystem with a few novel uses and explanations of the functionality being shared in articles and talks.

Right now, I'd argue that Async Hooks is in a similar adoption curve to that of Node.js itself. In that curve, we’re at the bleeding edge stage, where the extremely experienced developers who understand performance and monitoring in a way most others don't are beginning to play with Async Hooks.

The next stage of adoption will likely be individuals building tooling leveraging the work of those from the bleeding edge stage to expose the power of Async Hooks to the greater ecosystem for performance and monitoring of applications and code.

## Node.js 10 "Dubnium" LTS: Coming Soon

As per the [release schedule](https://github.com/nodejs/Release#release-schedule), Node.js 10 will become Node.js 10 "[Dubnium](https://en.wikipedia.org/wiki/Dubnium)" LTS in October.

This means that both the features shipping with Node.js 10 on release and any features that are developed and included prior to the LTS release date will be supported until April 2021.

## Just one more thing...

We deeply care about Node.js and the LTS releases at NodeSource, seeing it as a key point of stability for the entire Node.js ecosystem. We've built out an entire product–[N|Solid](https://nodesource.com/products/nsolid/technical-details)–on the active Node.js LTS lines, because of the stability and reliability they provide. We're excited to offer N|Solid + Node.js 10 once Node.js 10 goes LTS in October!

If you'd like to stay in the loop with the tools, tutorials, tips, and more around Node.js releases and community, be sure to follow [@NodeSource](https://twitter.com/NodeSource) on Twitter and keep an eye on the NodeSource Blog to keep up to date.
