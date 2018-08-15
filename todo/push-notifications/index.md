>原文地址：https://developers.google.com/web/fundamentals/push-notifications/index

>译文地址：

>译者：刘鹏

>校对者：

描述：推送通知是原生APP中最重要的能力之一。现在Web也有这个能力了。为了能让用户充分使用它们，通知必须能够达到及时、准确和相关。

# Web推送通知: 及时、相关和准确 

{% include "web/_shared/contributors/josephmedley.html" %}


<img src="images/cc-good.png" alt="Example Notification" class="attempt-right">

如果你问一屋子的开发者，有哪些功能是移动设备拥有，而Web缺失的。推送通知一定位居前列。

Web推送通知允许用户在他们喜欢的网站一有更新之后就选择参与进来。同时允许开发者使用自定义的以及和用户相关的内容来有效地吸引用户。

推送API和通知API给予了开发者一系列全新的可能性去和用户重修旧好。

## 和service worker相关吗?

是的，推送是基于 service worker 的，因为 service worker 在后台负责操作。这就是说只有用户点击或者关掉通知的时候，相关代码才会运行（换另一句话说就是电池被消耗）。如果你现在还对这个不熟悉，请查看 [service worker introduction](https://developers.google.com/web/fundamentals/primers/service-workers/) 章节。在后面的章节当中我们会使用 service worker 代码给大家展示如何实现推送和通知。

## 两种技术

推送和通知使用不同但是互补的API。[**推送**](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)在服务器提供给 service worker 信息的时候被调用。[**通知**](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)是 service worker 或者网页 script 展示信息给用户的一种方式。

## 对通知的一点剖析

在下面的章节当中，我们直接用代码来展示如何使用通知。在 service worker 注册完成之后，我们可以在注册成功的 service worker 对象上调用 `showNotification` 方法。

    serviceWorkerRegistration.showNotification(title, options);

`title` 参数表示的是通知的标题。`options` 参数是一个对象字面量，它用于设置通知的其它属性。options 对象通常表示如下：

    {
      "body": "Did you make a $1,000,000 purchase at Dr. Evil...",
      "icon": "images/ccard.png",
      "vibrate": [200, 100, 200, 100, 200, 100, 400],
      "tag": "request",
      "actions": [
        { "action": "yes", "title": "Yes", "icon": "images/yes.png" },
        { "action": "no", "title": "No", "icon": "images/no.png" }
      ]
    }

<img src="images/cc-good.png" alt="Example Notification" class="attempt-right">

这些代码将生成一个如图所示的通知。它通常会提供和原生应用一样的能力。在深入到实现这些能力的细节之前，我将给你们展示如何有效地使用它们。我们将继续讲述实现推送通知的机制，包括如何处理权限，订阅，发送消息，以及如何回应消息等方面。


## 我要怎样才能试用它呢?

在你完全了解它们是怎么运作或者你必须实现这些功能之前，你可以尝试下面几种方式来试用这些特性。

1. 第一个是，查看 [our own sample](https://github.com/GoogleChrome/samples/tree/gh-pages/push-messaging-and-notifications)；

2. 第二个是，Peter Beverloo 的 [Notification Generator](https://tests.peter.sh/notification-generator/)；

3. 第三个是，Mozilla 的 [Push Payload Demo](https://serviceworke.rs/push-payload_demo.html)。

提示: 除非你的页面是 localhost， 否则的话推送API必须要求页面是HTTPS的。

<<../../_common-links.md>>
