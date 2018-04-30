>原文地址：https://developers.google.com/web/fundamentals/push-notifications/index

>译文地址：

>译者：刘鹏

>校对者：

描述：推送通知是原生APP中最重要的能力之一。现在Web也有这个能力了。为了能让大家充分使用它们，通知必须能够达到及时、准确和相关。

# Web推送通知: 及时、相关和准确 

{% include "web/_shared/contributors/josephmedley.html" %}


<img src="images/cc-good.png" alt="Example Notification" class="attempt-right">

如果你问一屋子的开发者，有哪些功能是移动设备拥有，而Web缺失的。推送通知一定是其中排名非常高的那个。

有了Web推送通知，用户喜欢的网站一有更新之后，他们就可以选择参与进来。开发者还可以将根据用户订制的相关内容推送给他们。

推送API和通知API让开发者在与用户的参与上给予了一个全新的可能性。

## 和service-worker相关吗? {: #service-worker-involved }

是的，推送是基于service worker的，因为service worker在后台负责操作。这就是说只有用户点击或者关掉通知的时候，
相关代码才会运行（换另一句话说就是电池被消耗）。如果你现在还对这个不熟悉，请查看[service worker introduction][service-worker-primer]章节。
在后面的章节当中我们会使用service worker代码给大家展示如何实现推送和通知。

## 两种技术 {: #two-technologies }

推送和通知使用不同但是互补的API。[**推送**](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)在服务器提供给service worker信息
的时候被调用。[**通知**](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)是service worker或者网页script展示信息给
用户的一种方式。

## 对通知的一点剖析 {: #anatomy }

在下面的章节当中，我们直接用代码来展示如何使用通知。在service worker注册完成之后，我们可以在注册成功的service worker对象上调用`showNotification`的方法。

    serviceWorkerRegistration.showNotification(title, options);

`title`参数表示的是通知的标题。`options`参数是一个对象字面量，它用于设置通知的其它属性。options对象通常表示如下：

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

这些代码将生成和图片中一样的一个通知。它通常会提供和原生应用一样的能力。在深入到实现这些能力的细节之前，我将给你们展示如何
有效地使用这些能力。我们将继续回到讲述实现推送通知的机制，包括如何处理权限，订阅，发送消息，以及如何回应消息等这些方面。


## 我要怎样才能试用它呢?

在你完全了解它们是怎么运作或者你必须实现这些功能之前，有好几种方式让你可以试用这些特性。
第一个是，查看[our own sample](https://github.com/GoogleChrome/samples/tree/gh-pages/push-messaging-and-notifications)；
第二个是，Peter Beverloo的[Notification Generator](https://tests.peter.sh/notification-generator/)；
第三个是，Mozilla的[Push Payload Demo](https://serviceworke.rs/push-payload_demo.html).

提示: 除非你的页面是localhost, 否则的话推送API必须要求页面是HTTPS的。

<<../../_common-links.md>>
