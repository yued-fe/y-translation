>原文地址：https://developers.google.com/web/fundamentals/push-notifications/common-issues-and-reporting-bugs

>译文地址：

>译者：刘文涛

>校对者：

# Web Push: Common Issues and Reporting Bugs

# Web 推送：常见问题以及错误反馈

When you hit an issue with web push, it can be difficult to debug the issue or
find help. This doc outlines some of the common issues and what you should
do if you've found a bug in Chrome or Firefox.

当您使用网络推送遇到问题时，可能很难去调试这个问题或寻求帮助。 本文将概述了一些常见问题以及如果你在Chrome或Firefox中发现错误，应该怎么去做。

Before we dive into debugging push, you may be hitting issues with debugging
service workers themselves, the file not updating, failing to register or
generally just unusual behavior. There is an
[awesome document on debugging service workers](https://developers.google.com/web/fundamentals/web/fundamentals/getting-started/codelabs/debugging-service-workers/)
that I strongly recommend checking out if you are new to
service worker development.

在我们深入调试推送之前，您可能遇到 service workers 本身的问题，文件未更新，未注册或一般的异常行为。 关于[调试 service workers](https://developers.google.com/web/fundamentals/web/fundamentals/getting-started/codelabs/debugging-service-workers/) 的文档非常完善，如果你是初次使用 service worker 开发的话 ，我强烈建议你去阅读一下。

There are two distinct stages to check off when developing and testing web push,
each with their own set of common issues / problems.

在开发和测试Web推送时，我们会遇到是两个截然不同的阶段，每个阶段都有自己的常见问题。

- **Sending a Message:** Make sure that sending messages is successful.
   You should be getting a 201 HTTP code. If you aren't :
    - **Check for Authorization Errors:** If you receive an authorization
       error message see the
       [Authorization Issues section](#authorization_issues).
    - **Other API Errors:** If you receive a non-201 status code response,
       see the [HTTP Status Codes section](#http_status_codes) for
       guidance on the cause of the issue.
- **Receiving a Message**: If you're able to send a message successfully,
   but the message is not received on the browser:
    - **Check for Encryption Issues:** See the [Payload Encryption
       Issue Section](#payload_encryption_issue).
    - **Check for Connection Issues:** If the problem is on Chrome, it
       may be a connection. See [Connection Issues section](#connection_issue)
       for more info.
       
       
- **发送消息：**确保发送消息成功。 
	您应该获得201 HTTP状态码。 如果你不是：
	- **检查授权错误：**如果您收到授权错误消息，请参阅下面 “授权相关问题部分"。
	- **其他API错误：**如果收到非201状态代码，请参阅下面的 “HTTP状态代码部分“ 以获取有关问题原因的指导。
- **接收消息：**如果你能够成功发送消息，但在浏览器上未收到消息：
	- **检查加密问题：**请参下面的 “有效负载加密问题部分“。
	- **检查连接问题：**如果问题出在Chrome上，可能是连接。 可参阅下面的 “连接问题部分”。

If you aren't able to send and receive a push message and the relevant sections
in this doc aren't helping debug the problem then you may have found a
bug in the push mechanism itself. In this case, refer to the
[Raising Bug Reports](#raising_bug_reports)
section to file a good bug report with all the necessary information to expedite
the bug fixing process.

如果您无法发送和接收推送消息，并且本文档中的相关部分不能帮助你调试问题，那么你可能发现了推送机制本身的一个 bug。在这种情况下，请参阅 “如何提交错误报告 部分”，提交一份包含所有重要信息的错误报告，以加快错误修复过程。

One thing I'd like to call out before we start is that **Firefox and the
Mozilla AutoPush Service have great errors messages.** If you get stuck and
are not sure what the problem is, then test in Firefox and see if you
get a more helpful error message.

在开始之前我想说的一件事：**Firefox和Mozilla 自动推送服务给了很多有用的错误信息**。 如果您遇到问题并且不确定是什么问题的石斛，那么请在Firefox中进行测试，看看是否可以收到了更有用的错误消息。


## Authorization Issues

## 授权问题

Authorization issues are one of the most common issues developers hit when
starting out with web push. This is normally a problem with configuration of a
sites [Application Server Keys (a.k.a VAPID keys)
](https://tools.ietf.org/html/draft-ietf-webpush-vapid-02).

授权问题是开发人员在开始使用Web推送时遇到的最常见问题之一。 这通常是配置站点[应用服务器密钥（又名VAPID密钥）](https://tools.ietf.org/html/draft-ietf-webpush-vapid-02)的问题。

The easiest way to support push in both Firefox and Chrome is to supply an
`applicationServerKey` in the `subscribe()` call. The down side is that
any discrepancy between your front end and server's keys will result in an
authorization error.

在Firefox和Chrome中，支持推送的最简单方法是在 `subscribe()` 调用中提供 `applicationServerKey`。这样做不好的是，前端和服务器密钥之间的任何差异都会导致授权错误。

### On Chrome + FCM

### 在chrome + 云消息传递FCM

For Chrome, which uses FCM as a push service, you'll receive an
`UnauthorizedRegistration` response from FCM for a range of different
errors, all involving the application server keys.

对于使用FCM作为推送服务的Chrome，您将收到来自FCM 关于UnauthorizedRegistrationresponse【未经授权注册】 的一系列不同的错误，所有错误都涉及应用程序服务器密钥。

You'll receive an `UnauthorizedRegistration` error in any of the following
situations:

在以下任何一种情况下，您都会收到一个 `UnauthorizedRegistration` 错误：

* If you fail to define an `Authorization` header in the request to FCM.
* Your application key used to subscribe the user doesn't match the key used
  to sign the Authorization header.
* The expiration is invalid in your JWT, i.e. the expiration exceeds 24 hours or
  the JWT has expired.
* The JWT is malformed or has invalid values.

* 如果你没有对FCM的请求中定义Authorization header。
* 用于订阅用户的应用程序密钥与用于签署Authorization header的密钥不匹配。
* 您的JWT【Json web token】到期无效，即超过24小时或JWT已过期。
* JWT异常或值无效。

The full error response looks like this:

完整的错误响应如下所示：

```
<HTML>\n<HEAD>\n<TITLE>UnauthorizedRegistration</TITLE>\n</HEAD>\n<BODY BGCOLOR="#FFFFFF" TEXT="#000000">\n<H1>UnauthorizedRegistration</H1>\n<H2>Error 400</H2>\n</BODY>\n</HTML>\n
```
If you receive this error message in Chrome, consider testing in Firefox to see
if it'll provide more insight to the problem.

如果您在Chrome中收到此错误消息，可以考虑在Firefox中进行测试，可能能够提供有关此问题的更多信息的话。

### Firefox and Mozilla AutoPush

### Firefox and Mozilla 自动推送

Firefox and Mozilla AutoPush provides a friendly set of error messages for
`Authorization` issues.

Firefox和Mozilla 自动推送为授权问题提供了一组友好的错误提示。

You'll also receive an `Unauthorized` error response from
Mozilla AutoPush if the `Authorization` header is not included in your push
request.

如果您的推送请求中未包含Authorization header，你将会收到来自 Mozilla 自动推送的没有授权的错误信息。

```
{  
	"errno": 109,  
	"message": "Request did not validate missing authorization header",  
	"code": 401,  
	"more_info": "http://autopush.readthedocs.io/en/latest/http.html\#error-codes",  
	"error": "Unauthorized"  
}
```

If the expiration in your JWT has expired, you'll also receive an
`Unauthorized` error with a message that explains that the token has
expired.

如果JWT的已过期，您将会收到一条 `Unauthorized` 未授权的错误信息，该信息说明该令牌已过期。

```
{  
	"code": 401,  
	"errno": 109,  
	"error": "Unauthorized",  
	"more_info": "http://autopush.readthedocs.io/en/latest/http.html\#error-codes",  
	"message": "Request did not validate Invalid bearer token: Auth expired"  
}
```

If the application server keys are different between when the user was
subscribed and when the Authorization header was signed, a `Not Found`
error will be returned:

如果应用程序服务器密钥与订阅用户的授权标头签署时不同，则会返回未找到错误：

```
{  
	"errno": 102,  
	"message": "Request did not validate invalid token",  
	"code": 404,  
	"more_info": "http://autopush.readthedocs.io/en/latest/http.html\#error-codes",  
	"error": "Not Found"  
}
```

Lastly if you have an invalid value in your JWT (for example if the "alg" value
is an unexpected value) you'll receive the following error from Mozilla
AutoPush:

最后，如果你的的JWT中有无效值（例如，“alg”值是一个异常的值），您将会从Mozilla 自动推送中收到以下错误信息：

```
{  
	"code": 401,  
	"errno": 109,  
	"error": "Unauthorized",  
	"more_info": "http://autopush.readthedocs.io/en/latest/http.html\#error-codes",  
	"message": "Request did not validate Invalid Authorization Header"  
}
```

## HTTP Status Codes

## HTTP 状态码

There are a range of issues that can result in a non-201 response code from a
push service. Below is a list of HTTP status codes and what they mean in relation
to web push.

有一系列的问题可能导致推送服务的非201响应代码。 下面是相关HTTP状态码列表及其与Web推送相关的问题描述。

<table>
<tr>
<th>Status Code</th>
<th>Description</th>
</tr>
<tr>
<td>429</td>
<td>Too many requests. Your application server has reached a rate limit with a
push service. The response from the service should include a 'Retry-After' header to
indicate how long before another request can be made.</td>
</tr>
<tr>
<td>400</td>
<td>Invalid request. One of your headers is invalid or
poorly formatted.</td>
</tr>
<tr>
<td>404</td>
<td>Not Found. The subscription has expired. In this case you
should delete the PushSubscription from your back end and wait for an
opportunity to resubscribe the user.</td>
</tr>
<tr>
<td>410</td>
<td>Gone. The subscription is no longer valid and should be removed from your
back end. This can be reproduced by calling `unsubscribe()` on a
`PushSubscription`.</td>
</tr>
<tr>
<td>413</td>
<td>Payload size too large. The minimum size payload a push service must
support is 4096 bytes (or 4kb). Anything larger can result in this error.</td>
</tr>
</table>

<table>
<tr>
<th>Status Code</th>
<th>Description</th>
</tr>
<tr>
<td>429</td>
<td>请求太多。 您的应用程序服务器已通过推送服务达到了费率限制。 服务的响应应包括“Retry-After” header，设置需要多长时间的间隔另一个请求才能开</td>
</tr>
<tr>
<td>400</td>
<td>无效的请求。 你的某一个header无效或格式不正确。</td>
</tr>
<tr>
<td>404</td>
<td>没有找到。订阅已过期。在这种情况下,你应该删除 PushSubscription 从你的后台，并且等待一个时机再次给用户订阅。/td>
</tr>
<tr>
<td>410</td>
<td>失效. 订阅不再有效,应该从你的后台移除。这可以在 “PushSubscription” 上 通过调用 “unsubscribe()” 方法来移除。</td>
</tr>
<tr>
<td>413</td>
<td>有效载荷大小太大。 推送服务必须支持的最小大小有效负载是4096字节（或4kb）。 任何更大的大小都可能导致此错误。?</td>
</tr>
</table>

If the http status code is not in this list and the error message is not
helpful, check the [Web Push Protocol
spec](https://tools.ietf.org/html/draft-ietf-webpush-protocol) to see if the
status code is referenced along with a scenario of when that status code can
be used.

如果http状态码不在此列表中且错误消息无效，在状态码当前状态可以被使用的情况下，可以查看下 [Web Push Protocol
spec](https://tools.ietf.org/html/draft-ietf-webpush-protocol)规范 【Web推送协议规范】。

## Payload Encryption Issue

## 有效载荷加密问题

If you can successfully trigger a push message (i.e. send a message to a web
push service and receive a 201 response code) but the push event never fires in
your service worker, this normally indicates that the browser failed to
decrypt the message it received.

如果你可以成功触发推送消息（即向Web推送服务发送消息并接收到201响应码），但推送事件不会在 service worker 中触发，这通常表示浏览器无法解密其接收到的消息。

If this is the case, you should see an error message in Firefox's DevTools
console like so:

如果是这种情况，您应该可以在 Firefox 的 DevTools 控制台中看到一条错误消息，如下所示：

![Firefox DevTools with decryption message](./images/ff-devtools-decryption-msg.png)

To check if this is the issue in Chrome, do the following:

要检查Chrome中是否存在此问题，请执行以下操作：

1. Go to chrome://gcm-internals and click the "Start Recording" button.

1. 地址栏输入 chrome://gcm-internals ，进入并点击“Start Recording【开始录制】”按钮。

![Chrome GCM internals record](./images/gcm-internals-start-recording.png)

2. Trigger a push message, and look under the "Message Decryption Failure Log".

2. 触发一个推送消息,看下“消息解密失败日志”。

![GCM internals decryption log](./images/gcm-internals-decryption-log.png)

If there was an issue with the decryption of the payload, you'll see an error
similar to the one displayed above. (Notice the `AES-GCM decryption failed`
message in the details column.)

如果有效负载的解密存在问题，您将看到类似于上面显示的错误。 （请注意详细信息列中的`AES-GCM decryption failed`消息。）

There are a few tools which may help debug encryption if this is your issue:

如果这是这个问题，有一些工具可以帮助你调试加密：

* [Push Encryption Verifier tool by Peter
  Beverloo](https://tests.peter.sh/push-encryption-verifier/).
* [Web Push: Data Encryption Test Page by
  Mozilla](https://mozilla-services.github.io/WebPushDataTestPage/)
  
* [推送加密验证工具Peter Beverloo](https://tests.peter.sh/push-encryption-verifier/)
* [网络推送：Mozilla的数据加密测试页面](https://mozilla-services.github.io/WebPushDataTestPage/)

## Connection Issue

## 连接问题

If you aren't receiving a push event in your service worker and you aren't
seeing any decryption errors, then the browser may be failing to connect to
a push service.

如果你没有在 service worker 中收到推送事件，并且没有看到任何解密错误，可能是浏览器无法连接到推送服务。

In Chrome you can check whether the browser is receiving messages by examining
the 'Receive Message Log' (sic) in `chrome://gcm-internals`.

在Chrome中，您可以通过页面：chrome://gcm-internals 中的“接收消息日志”模块来检查浏览器是否正在接收消息。

![GCM internals receive message log](./images/gcm-internals-receive-log.png)

If you aren't seeing the message arrive in a timely fashion then make sure that
the connection status of your browser is `CONNECTED`:

如果您没有及时看到消息，请确保你的浏览器的连接状态为CONNECTED，如下图所示：

![GCM internals connection state](./images/gcm-internals-connection-state.png)

If it's **not** 'CONNECTED', you may need to delete your current profile and
[create a new one](https://support.google.com/chrome/answer/2364824). If that
still doesn't solve the issue, please raise a bug report as suggested below.

如果它不是“CONNECTED”，您可能需要删除当前的配置文件并创建一个新。 如果仍然无法解决问题，请按照下面章节的建议提出错误报告。

## Raising Bug Reports

## 错误反馈

If none of the above helps with your issue and there is no sign of what the
problem could be, please raise an issue against the browser you are having an
issue with:

如果上面的方式都不能解决你的问题，并且没有迹象表明问题可能是什么，请根据你遇到问题的浏览器提出问题：

For Chrome, you'd raise the issue here:
[https://bugs.chromium.org/p/chromium/issues/list](https://bugs.chromium.org/p/chromium/issues/list)  

For Firefox, you should raise the issue on:
[https://bugzilla.mozilla.org/](https://bugzilla.mozilla.org/)

对于Chrome，你可以在此处反馈问题：[https://bugs.chromium.org/p/chromium/issues/list](https://bugs.chromium.org/p/chromium/issues/list)  

对于 Firefox，你可以在此处反馈问题: [https://bugzilla.mozilla.org/](https://bugzilla.mozilla.org/)

To provide a good bug report you should provide the following details:
如何提供一个好的的错误报告，你应该做到以下几点：

* Browsers you've tested in (i.e. Chrome version 50, Chrome version 51, Firefox
  version 50, Firefox version 51).
* An example `PushSubscription` that demonstrates the problem.
* Include any example requests (i.e. content of network requests to a push
  service, including headers).
* Include any example responses from network requests as well.

* 你测试的浏览器版本（例如Chrome版本50，Chrome版本51，Firefox版本50，Firefox版本51）。
* 给出一个例子，可以重现这个问题；
* 报告中应该包括请求的所有信息（即对推送服务的网络请求的内容[包含header]）。
* 报告中应该包括网络请求的返回的响应数据。

If you can provide a reproducible example, either source code or a hosted web
site, it often speeds up diagnosing and solving the problem.

如果你能提供一个可重现的例子,或者源代码或托管网站,它经常可以帮助我们快速解决问题。

