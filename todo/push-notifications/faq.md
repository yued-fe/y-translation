>原文地址：https://developers.google.com/web/fundamentals/push-notifications/faq

>译文地址：

>译者：

>校对者：







# FAQ 

## 为什么推送在浏览器关闭的时候不工作

这个问题颇有争议，主要是因为有一些场景使得这个问题比较难以寻找原因和难以理解。

让我们先从 Android 开始。Android 系统被设计成监听推送消息，一旦收到一条消息，唤醒对应的 Android 应用来处理推送消息，而不管这个应用是否关闭。

Android 上的任何浏览器和上述应用表现一致，当接收到一条推送消息的时候，浏览器会被唤醒，然后浏览器再唤醒你的 service worker，发出推送事件。

桌面操作系统则有细微差别。Mac OS X 系统有一个可视化的标记来辅助解释不同的场景，是最容易理解的。

在 Mac OS X 系统上，你可以从 dock 中排列的应用 icon 下的标记来判断一个程序是否在运行。

如果你对比一下下面 dock 中的两个 Chrome icon，通过 icon 下面的标记我们可以知道左边的那个正在运行，然而右边的那个 Chrome **没有在运行**，因为缺少了下面的标记。

![ OS X 的示例](./images/faq/os-x-dock.png)

在桌面系统上接收推送消息的情况下，当浏览器在运行的时候（即在 icon 下面有一个标记），你可以接收到消息。

这就是说，即使浏览器窗口没有打开，你仍然可以在你的 service worker 当中接收到推送消息，这是因为浏览器正在后台运行。

推送不能接收到的唯一情况就是浏览器被完全关闭了，即完全不在运行（ icon 下没有标记）。这同样适用于 Windows 操作系统，虽然在 Windows 上判断 Chrome 是否在后台运行有一点复杂。

## 我如何让用户在点击推送消息后，能够全屏打开我添加到桌面的 Web App？

在 Android 系统的 Chrome 浏览器上，Web App 可以被添加到桌面。当它在桌面被打开的时候，可以以没有地址栏的全屏模式打开，就像下面显示的那样。

![桌面图标全屏方式打开](./images/faq/gauntface-homescreen-to-fullscreen.png)

为了保持体验的一致性，开发者也想用户在点击通知之后也全屏打开他们的 Web App。

Chrome 在某种程度上实现了这个特性，虽然你可能发现它不太靠谱并且难以寻找原因。相关的实现细节如下：

> 在 Android 系统上，添加到桌面的网站在点击推送消息的时候，应该被允许以 **standalone（独立）** 模式打开。然而即使网站被添加到了桌面，Chromium 依然不能检测到这些
> 网站是否在桌面上。我们启发式地采用了下面这种方法：那些在最近 10 天内从桌面启动过的网站，点击通知后将以 **standalone** 模式打开。
> --[Chrome Issue](https://bugs.chromium.org/p/chromium/issues/detail?id=541711)

这就意味着，除非你的用户足够频繁地通过桌面访问你的站点，否则的话你的通知只会打开一个普通的浏览器 UI。

这个问题将进一步解决。

**注意：** 这个只是 Chrome 的表现，其它浏览器也会有一些不同。如果你有任何需要讨论的，请尽管[提出 issue](https://github.com/gauntface/web-push-book/issues)

## 为什么这个比 web sockets 要好？

即使浏览器窗口是关闭的，service worker 依然可以工作。而 web socket 只有在浏览器和网页保持打开的状态下才能正常工作。

## GCM, FCM, Web 推送 和 Chrome 是怎么回事？

这个问题有很多方面，最简单的解释方法是逐步介绍 Web 推送和 Chrome 的历史。（别担心，很短）

##### 2014 年 12 月
当 Chrome 首次实现 Web 推送时，是使用 Google Cloud Messaging（GCM）来支持从服务器向浏览器发送推送消息的。

但这不是 **Web 推送**。早期的 Chrome 和 GCM 并不是“真正的” Web 推送有几个原因：

- GCM 要求开发人员在 Google Developers Console 上注册帐户。
- Chrome 和 GCM 需要一个特殊的由 Web 应用程序共享的发送者 ID，来正确地设置消息。
- GCM 的服务器接收的是自定义的 API 请求，而非 Web 标准。

##### 2016 年 7 月
7月，Web 推送中的一项新功能发布 - Application Server Keys（即规范中的 VAPID ）。Chrome 在支持此新 API 时，弃用了 GCM，改用 Firebase Cloud Messaging（FCM）作为消息传递服务。这很重要，原因如下：

- Chrome 和 Application Server Keys **不需要**使用 Google 或 Firebase 设置任何类型的项目，就可以正常工作。
- FCM 支持 **Web 推送协议**，这是所有 Web 推送服务都支持的 API。这意味着无论浏览器使用什么推送服务，你只需发送相同类型的请求，它就会推送消息。

##### 为什么现在令人困惑？
现在已经存在的关于 Web 推送的内容中存在大量混淆，其中大部分内容都用了 GCM 或 FCM。 如果内容用了 GCM，你可以将其视为是过时内容的标志，或者它过于关注 Chrome。（我在一些旧的文章中也犯了这个错误）

因此，你应该将 Web 推送视为浏览器的一部分，浏览器使用推送服务来管理发送和接收消息，其中推送服务将接受符合“ Web 推送协议”的请求。如果按照这样进行思考，你就可以无视浏览器及其使用的推送服务的干扰，并直接开始工作了。

本书的编写专注于 Web 推送的标准方法，所以故意忽略其他任何内容。

## Firebase 有一个 JavaScript SDK，它是什么以及为什么要有它？
对于那些已经发现 Firebase Web SDK 且注意到它有 JavaScript 版本的消息传递 API 的人，可能会想知道它与 Web 推送的区别。

消息传递SDK（Firebase Cloud Messaging JS SDK）在幕后做了一些工作以便更轻松地实现 Web 推送。

- 只需要关心 FCM 令牌（仅仅是一个字符串），而不必关心 `PushSubscription` 及其各个字段。
- 通过使用每个用户的令牌，你可以使用专有的 FCM API 来触发推送消息。此 API 不需要加密有效负载，可以在 POST 请求 body 中发送普通（未加密）的文本有效载荷。
- FCM 的专有 API 支持自定义功能，例如 [FCM 主题](https://firebase.google.com/docs/cloud-messaging/android/topic-messaging)（尽管相关文档很少，但它也可以在 Web 上运行）。
- 最后，FCM 支持 Android，iOS 和 Web，因此对于一些团队来说，在现有项目中更容易使用。

其实它在底层使用的还是 Web 推送，但是目的就是把 Web 推送抽象出来。

就像我在上一个问题中所说的那样，如果将 Web 推送视为浏览器加上推送服务，那么你可以将 Firebase 中 的 Messaging SDK 视为一个简化的 Web 推送的库。
