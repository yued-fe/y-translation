>原文地址：https://developers.google.com/web/fundamentals/push-notifications/subscribing-a-user

>译文地址：

>译者：

>校对者：

# 订阅一个用户


第一步是是从用户那里获取发送消息的权限，然后我们才能着手于"推送订阅"。

实现这个的Javascript API是相当直接，所以让我们来一步一步来看一下这个逻辑流程。

## 特征检测

第一步，我们需要检查用户当前的浏览器是否支持推送消息。我们可以通过下面两个简单的方法来检查推送是否支持。

1. 检查**navigator**上是否有**serviceWorker**属性。
2. 检查**window*上是否有*PushManager**属性。

```
if (!('serviceWorker' in navigator)) {
  // Service Worker isn't supported on this browser, disable or hide UI.
  return;
}

if (!('PushManager' in window)) {
  // Push isn't supported on this browser, disable or hide UI.
  return;
}
```

因为浏览器对service worker和推送消息两者的支持变得很快，对这两者同时进行特征检测并且进行[渐进增强](https://en.wikipedia.org/wiki/Progressive_enhancement)总是一个好主意。

## 注册一个service worker

通过特征检测，我们已经知道service worker和推送两者都已经支持了。下一步就是去注册我们的service worker。

当我们注册service worker的时候，我们就是告诉浏览器我们的service worker文件在哪里。这个文件依然是JavaScript，但是浏览器会给它访问系统service worker API的权限，包括推送。
更确切的说，浏览器是在service worker环境来运行这个文件。

通过调用`navigator.serviceWorker.register()`来注册一个service worker。传入参数是我们文件的路径。如下面所示：

    function registerServiceWorker() {
      return navigator.serviceWorker.register('service-worker.js')
      .then(function(registration) {
        console.log('Service worker successfully registered.');
        return registration;
      })
      .catch(function(err) {
        console.error('Unable to register service worker.', err);
      });
    }

上面的代码告诉浏览器，我们有一个service worker文件，并且告知了它的地址。在这个示例中，service worker文件地址是 "/service-worker.js"。
在调用完`register()`之后，在后台浏览器会进行下面几个步骤。

1. 下载service worker文件。

2. 运行JavaScript代码。

3. 如果一切都运行正常并且没有发生错误，调用 register() 之后返回的promise将会调用resolve方法。如果有任何错误发生，promise会reject。

> 如果 register() reject了，在Chrome的开发者工具当中再检查一遍你的JavaScript代码中的拼写错误或逻辑错误。

如果 register() 确实 resolve 了， 它会返回一个"ServiceWorkerRegistration"的方法。我们将使用它来访问[推送管理API](https://developer.mozilla.org/en-US/docs/Web/API/PushManager)

## 获取许可

我们注册了service worker，为订阅用户做好了准备，下一步就是从用户那里获取给他们发送消息的权限。

获取权限的API相对简单，下面就是那个API [最近变化为需要传入一个回调来返回一个Promise对象](https://developer.mozilla.org/en-US/docs/Web/API/Notification/requestPermission)
问题是我们不能分辨当前浏览器实现了哪一个API，所以我们必须同时实现两个。

    function askPermission() {
      return new Promise(function(resolve, reject) {
        const permissionResult = Notification.requestPermission(function(result) {
          resolve(result);
        });

        if (permissionResult) {
          permissionResult.then(resolve, reject);
        }
      })
      .then(function(permissionResult) {
        if (permissionResult !== 'granted') {
          throw new Error('We weren\'t granted permission.');
        }
      });
    }

在上面的代码当中，最重要的代码片段就是调用"Notification.requestPermission()"。这个方法会显示一个提示给用户：
![Permission Prompt on Desktop and Mobile Chrome.](./images/permission-prompt.png)

一旦这个权限被同意 / 允许，关闭（也就是点击弹层上的叉）或者被拦截，我们将获取结果字符串：'granted', 'default'或者'denied'。

在上面的示例代码中，如果权限被许可了，调用"askPermission()"返回的promise会resolve，否则的话我们会抛出一个错误让promise拒绝。

一个边界情况，我们一定要处理，那就是如果用户点击了'Block'的按钮。如果这个发生了，我们将再也不能够跟用户请求授权。他们必须手动地
"unblock"我们的应用，改变我们就用的权限状态，这个入口隐藏在浏览器的设置面板。仔细想想以什么方法以及什么时间向用户询问授权，因为如果他们点击block，再想让他们更改这个决定不是那么容易。

好消息是，只要他们知道为什么需要这个权限，大多数用户都很乐意授权给我们。

后续，我们将看一下一些流行的站点是怎么请求授权的。

## 使用PushManager订阅一个用户

一旦我们注册了service worker并且获得取权限，我们就能调用`registration.pushManager.subscribe()`订阅一个用户。

    function subscribeUserToPush() {
      return navigator.serviceWorker.register('service-worker.js')
      .then(function(registration) {
        const subscribeOptions = {
          userVisibleOnly: true,
          applicationServerKey: urlBase64ToUint8Array(
            'BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U'
          )
        };

        return registration.pushManager.subscribe(subscribeOptions);
      })
      .then(function(pushSubscription) {
        console.log('Received PushSubscription: ', JSON.stringify(pushSubscription));
        return pushSubscription;
      });
    }


当调用`subscribe()`方法的时候，我们传入一个*options*的对象，这个对象包含必须的和可选的参数。

让我们来看一下我们能传入的所有参数。

### 仅用户可见的选项

当推送一开始被添加到浏览器的时候，关于开发者是否能够发送消息给用户并且不显示通知，这一点是不确定的。这个一般被称为静默推送，这是因为用户不知道后台正在发生什么。

关键点就是开发者可能会做一些让人讨厌的事情，比如说持续不断地追踪用户的位置，而不让用户知道。

为了避免这个场景以及让规范作者们有时间来考虑如何更好地支持这个特性，他们添加了**userVisibleOnly**这个选项，并且和浏览器达成了一个象征性的协议，传入一个*true*的值，这样的话
每次收到一个推送，web应用都会展示出一个通知(也就是说没有静默推送）。

所以说当前，你**必须**传入一个**true**的值。如果你没有传入一个**userVisibleOnly**的键值或者传入的是*false*值，你会得到如下的错误：

> Chrome当前仅支持能够产生让用户可见消息的推送API的订阅。你可以调用`pushManager.subscribe({useVisibleOnly: true})。查看[https://goo.gl/yqv4Q4](https://goo.gl/yqv4Q4)获取更多详情。

当前看起来，在Chrome当中，完全的静默推送永远都不会被实现。规范作者们正在探索一个预算API的概念，它会基于用户对web app的使用而给开发者们一定量的静默推送消息次数。

### applicationServerKey 选项

在之前的章节当中，我们很明确地提到了"application server keys"，推送服务使用"application server keys"来用来鉴别订阅用户的应用，并且确保是同样的应用发送消息给那个订阅的用户。

"Application server keys"是一对公私钥的键值对，对于你的应用来说是独一无二的。私钥应该对你的应用保密，而公钥可以自由地分享。

传入到`subscribe()`方法的"applicationServerKey"选项应该是公钥。当订阅一个用户的时候，浏览器会将这个值传递给推送服务，这意味着推送服务可以将你的应用的公钥和用户的推送订阅绑定起来。

下面的图描述了这些步骤。

1. 浏览器加载了你的web app，然后你调用`subscribe()`，传入你的application server key中的公钥。
2. 然后浏览器发出一个网络请求到推送服务，推送服务会生成一个和applications public key联系在一起的endpoint，并把该 endpoint 返回给浏览器。
3. 浏览器将这个值添加到通过`subscribe()`返回的Promise对象 PushSubscription 当中。

![Illustration of the public application server key is used in subscribe
method.](./images/svgs/application-server-key-subscribe.svg)

当你后续想要发送一个推送消息，你将需要创建一个**Authorization**的header头，这个header头将包含使用你的application server的私钥签名的信息。
当推送服务接收到一个要求发送推送消息的请求，它通过查询关联到接收请求的这个 endpoint 的公钥，来验证这个签名过的**Authorization**的header头。如果签名是合法的，
推送服务知道它一定来自于拥有匹配的私钥的应用服务器。总的来说，它是一个安全措施，用来防止其他人发送消息给应用用户。

![How the private application server key is used when sending a
message.](./images/svgs/application-server-key-send.svg)

从技术上来说，`applicationSecretKey`是一个可选项。然页，在Chrome浏览器上最容易的实现是需要它的，其他浏览器在以后也可能需要它。在Firefox中当前是可选项。

在[VAPID spec](https://tools.ietf.org/html/draft-thomson-webpush-vapid)中定义了 application server key 的规范。
记住 application server key 和 VAPID keys 是同一个概念。

#### 如何创建一个Application server keys

你可以通过访问[web-push-codelab.glitch.me](https://web-push-codelab.glitch.me/)创建一个application server keys的公私钥。
或者你也可以使用[web-push command line](https://github.com/web-push-libs/web-push#command-line)通过下面几步来生成密钥。

    $ npm install -g web-push
    $ web-push generate-vapid-keys

你只需要为你的应用生成一次密钥，并且确保你把私钥保管好。（是的，我刚才说过这个）。

## 授权和订阅

在调用`subscribe()`时有一个副作用。就是你在调用`subscribe()`的时候，如果web应用没有获得弹出通知的许可，浏览器会为你请求许可。
如果你的UI和这个流程是匹配的，那这样的话是很有用的。但是如果你需要更多的控制（我认为绝大多数开发者都是这样想的），请使用我们之前用过的`Notification.requestPermission()`。

## 什么是推送订阅

我们调用`subscribe()`，传入一些选项，然后作为回报我们获得一个promise对象，这个对象resolve返回的就是推送订阅。相应的代码如下：

    function subscribeUserToPush() {
      return navigator.serviceWorker.register('service-worker.js')
      .then(function(registration) {
        const subscribeOptions = {
          userVisibleOnly: true,
          applicationServerKey: urlBase64ToUint8Array(
            'BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U'
          )
        };

        return registration.pushManager.subscribe(subscribeOptions);
      })
      .then(function(pushSubscription) {
        console.log('Received PushSubscription: ', JSON.stringify(pushSubscription));
        return pushSubscription;
      });
    }

这个推送订阅对象包含发送推送消息给那个用户所需要的所有信息。如果使用`JSON.stringify()`来打印内容，你可以看到如下所示：

    {
      "endpoint": "https://some.pushservice.com/something-unique",
      "keys": {
        "p256dh":
    "BIPUL12DLfytvTajnryr2PRdAgXS3HGKiLqndGcJGabyhHheJYlNGCeXl1dn18gSJ1WAkAPIxr4gK0_dQds4yiI=",
        "auth":"FPssNDTKnInHVndSTdbKFw=="
      }
    }

`endpoint`值就是推送服务的URL。如果要触发一个推送消息的话，向这个URL发送一个POST请求。

`keys`对象包含用于随着推送消息发送的消息数据被加密之后的值。

## 发送订阅到你的服务器

一旦你有了一个推送订阅，你将想要把它发送到你的服务器。怎么来做完全由你，但是一个小提示就是使用`JSON.stringify()`来从订阅对象当中获取所有的必需数据。
同样地你也可以手动地像这样拼凑成相同的结果：

    const subscriptionObject = {
      endpoint: pushSubscription.endpoint,
      keys: {
        p256dh: pushSubscription.getKeys('p256dh'),
        auth: pushSubscription.getKeys('auth')
      }
    };

    // 上面和下面的输出是一样的
    const subscriptionObjectToo = JSON.stringify(pushSubscription);

在web页面当中，像下面一样完成一个订阅的发送：

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

node服务接收到这个请求之后，保存数据到数据库当中供以后使用。

    app.post('/api/save-subscription/', function (req, res) {
      if (!isValidSaveRequest(req, res)) {
        return;
      }

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
    });

我们的服务器有了推送订阅的详细信息，我们就可以在任何我们想要的时候给我们的用户发送一条消息了。

## 常见问题解答

当前人们经常问的问题如下：

> 我可以更换浏览器使用的推送服务吗？

不行。推送服务是由浏览器选择的。正如我们看到的，当我们调用`subscribe()`时，浏览器会产生一个发送给推送服务的网络请求，来获取组成推送订阅的细节信息。

> 每个浏览器都使用不同的推送服务，那它们会有不同的API吗？

所有的推送服务拥有相同的API。

这个相同的API被称为[网络推送协议](https://tools.ietf.org/html/draft-ietf-webpush-protocol)
它描述了你的应用需要怎样的网络请求来触发一个推送消息。

> 如果用户在桌面版进行了订阅，那他们是不是同时在他们的手机版也订阅了？

不幸的是，没有。一个用户必须在他想要接收消息的所有浏览器都进行注册推送。值得注意的是，用户需要在每一个设备上都进行授权。
