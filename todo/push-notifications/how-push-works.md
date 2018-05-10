>原文地址：https://developers.google.com/web/fundamentals/push-notifications/how-push-works

>译文地址：

>译者：刘鹏

>校对者：


# 推送是怎么工作的?

在我们接触这个API之前，让我们先从一个高水平从头到尾来看一下`推送`。稍后我们通过逐一介绍各个主题和API让你知道为什么`推送`是这么的重要。

下面将介绍实现`push`的三个重要的步骤：

1. 添加客户端侧的逻辑来给用户订阅推送（也就是使用Web app当中的Javascript和UI，帮助用户注册推送消息）。
2. 从你的后台/应用调用API来触发推送消息到用户的设备。
3. 当推送到达用户的设备，service worker的Javascript将接收到 `推送事件`。在这个Javascript当中，你将能够展示出一个通知。

让我们更加详细的来看一下这三个步骤。

## Step 1: 客户端侧

第一步是`注册`一个用户来推送消息。

注册一个用户需要2个条件。第一，从用户那里获取给他们发送消息的**许可**。第二，从浏览器那里获取 `推送的订阅`。

`推送的订阅`包括我们需要推送消息给那个用户的所有信息。这个信息你可以部分认为是用户设备的ID。

这些全部可以通过Javascript使用[Push API](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)完成。

在我们订阅一个用户之前，我们需要生成一套应用器服务密钥。这个我们后续会解释。

应用服务器密钥，也被称为VAPID密钥，对你的服务器来说是独一无二的。有了它，推送服务知道哪一个应用服务器注册了一个用户，并且确保了是同一个服务器触发了推送消息给那个用户。

一旦你注册了一个用户，并且拿到了`PushSubscription`，你需要将关于这个`PushSubscription`的详细信息发送至你的后台或服务器。在服务器上，你需要将这个订阅保存至数据库当中。你可以使用这个订阅信息给那个用户推送消息。

![确保你发送了`PushSubscription`到你的后端](./images/svgs/browser-to-server.svg)

## Step 2: 发送一个推送消息

当你想要发送一个推送消息到你的用户，你需要执行一个API调用到推送服务。这个API包括：发送的数据，发送的对象和任何关于如何发送这条消息的标准。一般情况下这个API调用是由你的服务器来完成的。

你可能会有一些疑问：

- 推送服务是谁/什么？

- 这个API长什么样？它是JSON格式，XML格式，还是其他什么格式？

- 这个API能干什么？

### 推送服务是谁/什么?

推送服务接收到一个网络请求，校验该请求的正确性，然后发送一个推送消息到对应的浏览器。如果浏览器正好离线，这条消息会排队直到这个浏览器重新在线，然后发送给它。

每个浏览器都能他们想用的任何一个推送服务，这个是开发者管控不了的。这其实并不是一个问题，因为每一个推送服务都接受**相同**的API调用。也就是说你不必担心这个推送服务是谁。你只需要确保你的API调用是合法的。

要获得合适的触发推送消息的URL（也就是推送服务的URL），你只需要查看一下前面获得的`pushSubscription`的`endpoint`的值。

下面是`pushSubscription`的一个示例：

	{
	  "endpoint": "https://random-push-service.com/some-kind-of-unique-id-1234/v2/",
	  "keys": {
	    "p256dh" :
	"BNcRdreALRFXTkOOUHK1EtK2wtaz5Ry4YfYCA_0QTpQtUbVlUls0VJXg7A8u-Ts1XbjhazAkj7I99e8QcYP7DkM=",
	    "auth"   : "tBHItJI5svbpez7KI4CCXg=="
	  }
	}
	
这个示例当中的**endpoint**是*https://random-push-service.com/some-kind-of-unique-id-1234/v2/*。推送服务应该是'random-push-service.com'，每个endpoint对用户来说都是独一无二的。
即'some-kind-of-unique-id-1234'所代表的一样。当你开始着手于推送之后，你会注意到这一点。

关于上述示例当中的**key**这个字段，我们后续会讲到，这里就先不解释了。

### 这个API是什么?

我前面提到每个Web推送服务都需要的是相同的API调用。这个API就是[**Web Push Protocol**](https://tools.ietf.org/html/draft-ietf-webpush-protocol)。
它是一个IETF标准，它定义了你要如何向一个推送服务执行一个API调用。

这个API调用需要设置一些头部，并且需要以字节流的方式发送数据。我们将看一下如何用库来执行这个API，以及我们自己如何来实现这个API。

### 这个API能做什么？ 

这个API提供了一种发送消息到用户的一种方式，消息可以携带，也可以不携带数据。同时它也提供了*如何*发送消息的说明。


你通过推送消息发送的数据必须是加密的。原因是推送服务可能是任何人，你通过他来发送的数据必须防止他能够看到数据明文。
这很重要，因为是浏览器决定（而不是开发者）该用哪一个推送服务，而那些不安全的推送服务可能打开通往浏览器的大门。

当你触发一个推送消息，推送服务接收了API调用，然后将消息放到队列当中。这个消息会一直呆在队列当中，直到用户的设备上线，然后
推送服务就可以将消息发送过去。通过下面的说明你可以知道推送消息在推送服务当中是怎么排队的。

这些说明详情如下：

- 推送消息的TTL(生存时间)。这个定义了一条消息在没有发送并移除前，能在队列当中排多久的队。

- 设置消息的紧急度。这在推送服务为了保持用户的电量情况下，只推送高优先级的消息时很有用。

- 给推送消息一个话题名，它将用这个新消息替换任何其他排队的消息。

![When your server wishes to send a push message, it makes a web push protocol request to a
push service.](./images/svgs/server-to-push-service.svg)

## Step 3: 用户设备上的推送事件

一旦我们发出一个推送消息，推送服务会保留我们的消息，直至下面的任一一种情况发生：

1. 设备在线，然后推送服务发送消息。

2. 消息过期。如果消息过期，推送服务会将这条消息从它的队列当中移除，这样消息就永远不会再发送了。

当推送服务确实发送了一条消息，浏览器会接收到这条消息，解密数据，并且会在你的`service worker`当中发出一个`push`的事件。

[service worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)是一个特殊的JS文件。
即使你的页面没有找到，浏览器依然可以执行这个JS文件。甚至当浏览器关闭了的时候，这个JS也可以被执行。`service worker`也拥有它自己的API，比如`push`，
这些API在Web页面是不能被调用的（也就是说这些API不能在service worker脚本之外的地方被调用）

就是`service worker`的这个push事件让你能执行任何后台任务。你可以执行分析调用，缓存离线页面和弹出通知等。

![When a push message is sent from a push service to a user's device, your service worker
receives a push event.](./images/svgs/push-service-to-sw-event.svg)

这就是整个推送消息的流程。让我们更详细的来看一下其中的每一步吧。
