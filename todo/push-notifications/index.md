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

The Push API and Notification API open a whole new set of possibilities for
you to re-engage with your users.

## Are service workers involved? {: #service-worker-involved }

Yes. Push is based on service workers because service workers operate in the
background. This means the only time code is run for a push notification (in
other words, the only time the battery is used) is when the user interacts with
a notification by clicking it or closing it.   If you're not familiar with them,
check out the  [service worker introduction][service-worker-primer]. We will use
service worker code in later sections when we show you how to implement pushes
and notifications.

## Two technologies {: #two-technologies }

Push and notification use different, but complementary, APIs:
[**push**](https://developer.mozilla.org/en-US/docs/Web/API/Push_API) is
invoked when a server supplies information to a service worker; a
[**notification**](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
is the action of a service worker or web page script showing information
to a user.

## A little notification anatomy {: #anatomy }

In the next section we're going to throw a bunch of pictures at you, but we
promised code. So, here it is. With a service worker registration you call
`showNotification` on a registration object.


    serviceWorkerRegistration.showNotification(title, options);


The `title` argument appears as a heading in the notification. The `options`
argument is an object literal that sets the other properties of a notification.
A typical options object looks something like this:


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

This code produces a notification like the one in the image. It generally
provides the same capabilities as a native application. Before diving into the
specifics of implementing those capabilities, I'll show you how to use those
capabilities effectively.   We'll go on to describe the mechanics of
implementing push notifications, including handling permissions and
subscriptions, sending messages, and responding to them.

## How can I try it?

There are several ways you can play with the features before you completely
understand how they work, or have to implement them. First, check out
[our own sample](https://github.com/GoogleChrome/samples/tree/gh-pages/push-messaging-and-notifications).
Also available are Peter Beverloo's
[Notification Generator](https://tests.peter.sh/notification-generator/)
and Mozilla's [Push Payload Demo](https://serviceworke.rs/push-payload_demo.html).

Note: Unless you're using localhost, the Push API requires HTTPS.

<<../../_common-links.md>>
