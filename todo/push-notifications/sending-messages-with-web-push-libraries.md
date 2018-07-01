>原文地址：https://developers.google.com/web/fundamentals/push-notifications/sending-messages-with-web-push-libraries

>译文地址：

>译者：

>校对者：

# Sending Messages with Web Push Libraries 






开发网页推送的痛点之一就是触发一个推送消息是极其"繁琐"的，应用程序需要按照[ Web推送协议](https://tools.ietf.org/html/draft-ietf-webpush-protocol)向推送服务发送POST请求。为了使推送能够跨浏览器使用，你还需要使用 [VAPID](https://tools.ietf.org/html/draft-thomson-webpush-vapid)
(即应用服务器秘钥)——需要在 header 中设置一个值来证明你的应用能够向用户发送消息。发送推送消息数据时，需要对数据进行![加密](https://tools.ietf.org/html/draft-ietf-webpush-encryption)并添加特定的headers，以便浏览器正确地解密消息。

触发推送的主要问题是，如果遇到问题，很难进行诊断。随着时间的推移和更多浏览器的支持，这一点正在得到改善，但仍然并不容易。因此，我强烈推荐使用库来处理推送的加密、格式化、触发这一系列流程。

如果你想要深入学习这个库，我们会在下一个章节中介绍。现在，我们将着眼于管理订阅，并且使用现有的Web推送库来发送推送请求。

在这个章节，我们将使用[web-push Node
library](https://github.com/web-push-libs/web-push)。其他语言会有差异，但不会差太多。我们正在研究 node，因为它是 JavaScript，应该是读者最容易理解的。

注：如果你想要其他语言的库，可以查看 [web-push-libs organization
on Github](https://github.com/web-push-libs/).

我们将完成以下步骤:

1. 向我们的后端发送订阅并保存。
2. 检索保存的订阅并触发推送消息。

## 保存订阅

从数据库中保存并检索 `PushSubscriptions` 将取决于你的服务端语言和数据库选择，但查看如何完成这个步骤的示例可能很有用。

在 demo 页面中，通过发送简单的 POST 请求，`PushSubscription` 被发送到我们的后端:

```
function sendSubscriptionToBackEnd(subscription) {
  return fetch('/api/save-subscription/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(subscription)
  })
  .then(function(response) {
    if (!response.ok) {
      throw new Error('Bad status code from server.');
    }
    return response.json();
  })
  .then(function(responseData) {
    if (!(responseData.data && responseData.data.success)) {
      throw new Error('Bad response from server.');
    }
  });
}
```

demo 中的[Express](http://expressjs.com/)服务器会监听 `/api/save-subscription/` 路径:

```    app.post('/api/save-subscription/', function (req, res) {```

在这个路由下，我们会验证订阅以确保请求正确并且内容有效：

```
const isValidSaveRequest = (req, res) => {
  // Check the request body has at least an endpoint.
  if (!req.body || !req.body.endpoint) {
    // Not a valid subscription.
    res.status(400);
    res.setHeader('Content-Type', 'application/json');
    res.send(JSON.stringify({
      error: {
        id: 'no-endpoint',
        message: 'Subscription must have an endpoint.'
      }
    }));
    return false;
  }
  return true;
};
```

注：在这个路由中，我们只检查路径，如果你需要检验**必需**载荷，确保你也检查了 auth 和 p256dh 秘钥。

如果这个订阅是有效的，我们需要将其保存并返回一个合适的 JSON 响应:

```
return saveSubscriptionToDatabase(req.body)
.then(function(subscriptionId) {
  res.setHeader('Content-Type', 'application/json');
  res.send(JSON.stringify({ data: { success: true } }));
})
.catch(function(err) {
  res.status(500);
  res.setHeader('Content-Type', 'application/json');
  res.send(JSON.stringify({
    error: {
      id: 'unable-to-save-subscription',
      message: 'The subscription was received but we were unable to save it to our database.'
    }
  }));
});
```

这个 demo 使用了 [nedb](https://github.com/louischatriot/nedb) 存储订阅数据，这是一个简单的基于文件的数据库，你可以选择其他数据库，我们使用它仅是因为它支持零配置使用。在生产环境，你应该使用可靠性更高的数据库（我倾向使用旧版本的MySQL）。

```
function saveSubscriptionToDatabase(subscription) {
  return new Promise(function(resolve, reject) {
    db.insert(subscription, function(err, newDoc) {
      if (err) {
        reject(err);
        return;
      }

      resolve(newDoc._id);
    });
  });
};
```

## 发送推送请求

当发送推送消息时，我们最终需要一些事件来触发推送消息的流程。常用的方法是创建一个管理员页面，让你配置并触发消息推送。你也可以创建一个跑在本地的程序或者其他任何方法来访问 `PushSubscriptions` 列表、触发消息推送。

我们的 demo 有一个"类管理系统"的页面能够触发一个推送，因为是演示版本，所以这个页面是公开的。

我将演示开发这个 demo 所涉及的每个步骤，这是每个人都应该遵循的小步骤，包括任何刚接触Node的人。

在前文讨论订阅用户时，我们介绍了在 `subscribe()` 选项中添加 `applicationServerKey`，后端会需要这个私钥。



在 demo 中，这些值会被添加到我们的 Node 应用中，如下（我知道这段代码很无聊，但此处没有魔法）：

```
const vapidKeys = {
  publicKey:
'BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U',
  privateKey: 'UUxI4O8-FbRouAevSmBQ6o18hgE4nSG3qwvJTfKc-ls'
};
```

下一步，我们需要在 Node 服务器中安装 `web-push` 模块:

```
npm install web-push --save
```

然后在我们的 Node 脚本中引用 `web-push` 模块，如下：

```
const webpush = require('web-push');
```

现在我们可以使用 `web-push` 模块了。首先我们需要将应用服务器的秘钥（记住它们同时也是 VAPID 秘钥，这才是规范的命名）传给 `web-push` 模块。

```
const vapidKeys = {
  publicKey:
'BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U',
  privateKey: 'UUxI4O8-FbRouAevSmBQ6o18hgE4nSG3qwvJTfKc-ls'
};

webpush.setVapidDetails(
  'mailto:web-push-book@gauntface.com',
  vapidKeys.publicKey,
  vapidKeys.privateKey
);
```

我们也添加了一个 "mailto:" 字符串，这个字符串需要是一个 URL 或邮箱地址。这部分信息实际上会被作为触发推送请求的一部分发送给推送服务器。这么做的原因是，如果网络推搡服务需要与消息发送者联系，这些信息就能派上用场。

通过上述步骤，`web-push` 模块就可以使用了，下一步是触发一个消息推送。

这个 demo 使用了一个伪管理面板来触发消息推送。

![管理页面截图](./images/demo-admin-page.png)

点击"触发消息推送"将会给 `/api/trigger-push-msg/` 接口发送一个 POST 请求，相当于给后端一个信号去推送消息。所以我们需要在 express 中创建这个路径：

```
app.post('/api/trigger-push-msg/', function (req, res) {
```

当这个请求被收到时，我们会抓取并遍历数据库中的订阅信息，然后推送消息。

```
return getSubscriptionsFromDatabase()
.then(function(subscriptions) {
  let promiseChain = Promise.resolve();

  for (let i = 0; i < subscriptions.length; i++) {
    const subscription = subscriptions[i];
    promiseChain = promiseChain.then(() => {
      return triggerPushMsg(subscription, dataToSend);
    });
  }

  return promiseChain;
})
```

方法 `triggerPushMsg()` 能够使用 web-push 库来给订阅者发送消息。

```
const triggerPushMsg = function(subscription, dataToSend) {
  return webpush.sendNotification(subscription, dataToSend)
  .catch((err) => {
    if (err.statusCode === 410) {
      return deleteSubscriptionFromDatabase(subscription._id);
    } else {
      console.log('Subscription is no longer valid: ', err);
    }
  });
};
```

调用 `webpush.sendNotification()` 方法会返回一个 promise 对象。 如果这个消息发送成功，promise 会回调 resolve 函数，这时我们不用做其他事情。但当 promise 的回调 reject，你需要检验错误信息，它会告诉你 `PushSubscription` 是否仍然有效。

要确定推送服务的错误类型，最好的方法是查看状态码。错误消息因推送服务而异，不一定都有帮助。

在这个例子中，我们检验了状态码'401'和'402'，分别是 HTTP 状态码中的 'Not Fount（资源未找到）' 和 'Gone（资源不再可用）' ，收到这两个状态码意味着订阅过期或失效，我们需要将订阅信息从数据库中移除。

下个章节中我们将会更仔细地介绍 Web 推送协议以及一些其他状态码。

注：如果你在这个步骤遇到了问题，推荐去 Firefox 上查看错误日志而不是 Chrome。因为相比于 Chrome / FCM，Mozilla 的推送服务提供的错误信息更加有用。

遍历完订阅后，我们需要返回一个 JSON 响应。

```
.then(() => {
  res.setHeader('Content-Type', 'application/json');
    res.send(JSON.stringify({ data: { success: true } }));
})
.catch(function(err) {
  res.status(500);
  res.setHeader('Content-Type', 'application/json');
  res.send(JSON.stringify({
    error: {
      id: 'unable-to-send-messages',
      message: `We were unable to send messages to all subscriptions : ` +
        `'${err.message}'`
    }
  }));
});
```

至此，我们已经完成了主要的实现步骤。

创建一个订阅请求 API，让前端能够向后端发送一个订阅请求，并将订阅信息保存在数据库中。
2. 创建一个发送推送消息的 API（这一次请求需要从管理面板发出）。
3. 后端读取所有的订阅，并选择一个 [web-push
库](https://github.com/web-push-libs/)给每一个订阅发送消息。

无论你的后端使用什么语言 (Node, PHP, Python, ...) ，实现推送的步骤都是一样的。

接下来，这些 web-push 库究竟能为我们做了什么呢？