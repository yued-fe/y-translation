>原文地址：https://developers.google.com/web/fundamentals/push-notifications/common-notification-patterns

>译文地址：https://github.com/yued-fe/y-translation/edit/master/todo/push-notifications/common-notification-patterns.md

>译者：[任家乐](https://github.com/jennyrenjiale)

>校对者：[刘文涛](https://github.com/HSDPA-wen)、[杨芯芯](https://github.com/y2x33)

# 常用的通知模式

此篇我们将会探索Web推送的一些常用模式，包括使用一些service worker提供的API。

## 通知的关闭事件

在上一篇中，我们了解了如何监听`notificationclick`事件。

除了`notificationclick`事件，我们还可以监听`notificationclose`事件，它会在用户忽略其中一个通知（例如，用户点击了关闭按钮或划掉了通知，而不是点击了它）时被调用。

这个事件通常被用作数据分析，以此追踪用户与通知的互动情况。

    self.addEventListener('notificationclose', function(event) {
      const dismissedNotification = event.notification;

      const promiseChain = notificationCloseAnalytics();
      event.waitUntil(promiseChain);
    });


## 给通知添加数据

当收到推送的信息时，通常只需要获取用户点击后的有用数据。例如，获取用户点击通知时打开的页面地址。

如果需要将推送事件中获取的数据传递给通知，最简单的方式就是在调用`showNotification()`时，给参数options对象添加一个`data`属性，其值为对象类型，例如以下所示：

        const options = {
          body: 'This notification has data attached to it that is printed ' +
            'to the console when it\'s clicked.',
          tag: 'data-notification',
          data: {
            time: new Date(Date.now()).toString(),
            message: 'Hello, World!'
          }
        };
        registration.showNotification('Notification with Data', options);

在点击事件的回调内，可以通过`event.notification.data`来获取数据，例如：

      const notificationData = event.notification.data;
      console.log('');
      console.log('The data notification had the following parameters:');
      Object.keys(notificationData).forEach((key) => {
        console.log(`  ${key}: ${notificationData[key]}`);
      });
      console.log('');


## 打开一个窗口

对一个通知来说，打开指定地址的窗口／标签页可以说是一种最常见的反馈，这个我们可以通过[clients.openWindow()](https://developer.mozilla.org/en-US/docs/Web/API/Clients/openWindow)来实现。

在`notificationclick`事件中，我们会运行类似下面的代码来实现以上需求：

      const examplePage = '/demos/notification-examples/example-page.html';
      const promiseChain = clients.openWindow(examplePage);
      event.waitUntil(promiseChain);

在下一节中，我们会看下如何检测用户点击通知后跳转的页面是否已被打开，如果已被打开，我们可以直接呼起已打开的标签页，而不是打开一个新的标签页。

## 呼起一个已打开的窗口

如果可能，我们应该呼起一个已打开的窗口，而不是在每次用户点击通知时都打开一个新的窗口。

在我们探索如何实现之前，值得提醒的是**你只能够在与通知同域名的页面实现这个需求**。因为我们只能检测我们自己站点的页面是否已被打开，这也避免了
开发者看到用户正在浏览的所有站点。

再来看下之前的例子，我们会对代码稍作调整来检测页面'/demos/notification-examples/example-page.html'是否已经被打开。

      const urlToOpen = new URL(examplePage, self.location.origin).href;

      const promiseChain = clients.matchAll({
        type: 'window',
        includeUncontrolled: true
      })
      .then((windowClients) => {
        let matchingClient = null;

        for (let i = 0; i < windowClients.length; i++) {
          const windowClient = windowClients[i];
          if (windowClient.url === urlToOpen) {
            matchingClient = windowClient;
            break;
          }
        }

        if (matchingClient) {
          return matchingClient.focus();
        } else {
          return clients.openWindow(urlToOpen);
        }
      });

      event.waitUntil(promiseChain);

让我们逐步浏览下代码。

首先，我们将示例中目标页面的地址传递给URL API。这是我从[Jeff Posnick](https://twitter.com/jeffposnick)那学到的一个巧妙的计策。
调用`new URL()`并传入location对象，如果传入的第一个参数是相对地址，则会返回页面的绝对地址（例如，“/” 会变成 “https://站点域名” ）。

我们将地址转成了绝对地址则是为了之后与窗口的地址作对比。

      const urlToOpen = new URL(examplePage, self.location.origin).href;

之后，我们会通过调用`matchAll()`得到一系列`WindowClient`对象，包含了当前打开的标签页和窗口。（记住，这些标签页只是你域名下的页面）

      const promiseChain = clients.matchAll({
        type: 'window',
        includeUncontrolled: true
      })

`matchAll`方法中传入的options对象则告诉浏览器我们只想获取 “window” 类型的对象（例如，只查看标签页、窗口，不包含web workers[浏览器的其他工作线程]）。
`includeUncontrolled`属性表示我们只能获取没有被当前service worker控制的所有标签页（本域下），例如service worker正在运行当前代码。一般来说，在调用`matchAll()`时，你通常会将`includeUncontrolled`设置为true。

我们以`promiseChain`（promise链式调用）的形式捕获返回的promise对象，因此之后可以将其传入`event.waitUntil()`方法中以此保持我们的service worker持续工作。

当上一步的`matchAll()`返回的promise对象已完成异步操作，我们就可以开始遍历返回的window对象，并将这些对象的URL和想要打开的目标URL进行对比，如果发现有匹配的，则调用`matchingClient.focus()`方法，它会呼起匹配的窗口，引起用户的注意。

如果没有与之匹配的URL，我们则采用上一节的方式新开窗口打开地址。

      .then((windowClients) => {
        let matchingClient = null;

        for (let i = 0; i < windowClients.length; i++) {
          const windowClient = windowClients[i];
          if (windowClient.url === urlToOpen) {
            matchingClient = windowClient;
            break;
          }
        }

        if (matchingClient) {
          return matchingClient.focus();
        } else {
          return clients.openWindow(urlToOpen);
        }
      });

**注意：** 我们会返回`matchingClient.focus()`、`clients.openWindow()`方法执行后返回的promise对象，
这样promise对象就可以组成我们的promise调用链了。

## 合并通知

我们已经看到，给一个通知添加标签后会导致用同一个标签标识的已有通知被替代。

但通过使用通知相关的API，你可以更灵活地覆盖展示通知。比如一个聊天应用，开发者可能更希望用新的通知来展示"你有2条未读信息"等类似信息，而不是只展示最新接收到的信息。

你可以利用新的通知，或以其他方式操作当前已有通知，使用[registration.getNotifications()](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration/getNotifications)API能够获得到你APP中所有当前展示的通知。

让我们看看如何使用这个API去实现刚说的聊天应用的例子。

在聊天应用中，我们假设每个通知都有一些包含用户名的数据。

我们要做的第一件事就是在所有已打开的通知中找到带有具体用户名的用户。首先调用`registration.getNotifications()`方法，之后遍历其结果检测`notification.data`中是否有具体用户名。

        const promiseChain = registration.getNotifications()
        .then(notifications => {
          let currentNotification;

          for(let i = 0; i < notifications.length; i++) {
            if (notifications[i].data &&
              notifications[i].data.userName === userName) {
              currentNotification = notifications[i];
            }
          }

          return currentNotification;
        })

下一步就是用新的通知来替换上一步中获得的通知。

在这个虚拟的消息应用中，我们会给新的通知添加一个累计新通知数量的数据，每产生新的通知都会累加这个计数，以此来记录用户收到的新信息的数量。

        .then((currentNotification) => {
          let notificationTitle;
          const options = {
            icon: userIcon,
          }

          if (currentNotification) {
            // 我们有一个已经打开的通知，让我们利用它来做些什么
            const messageCount = currentNotification.data.newMessageCount + 1;

            options.body = `You have ${messageCount} new messages from ${userName}.`;
            options.data = {
              userName: userName,
              newMessageCount: messageCount
            };
            notificationTitle = `New Messages from ${userName}`;

            // 记得关闭旧的通知
            currentNotification.close();
          } else {
            options.body = `"${userMessage}"`;
            options.data = {
              userName: userName,
              newMessageCount: 1
            };
            notificationTitle = `New Message from ${userName}`;
          }

          return registration.showNotification(
            notificationTitle,
            options
          );
        });


我们会累加当前展示的通知的信息数，同时依据这个数据来设置通知的主题和内容信息。如果当前没有展示通知，我们则会展示一个新的通知，其数据中`newMessageCount`的值为1。

那么第一条信息的通知会是以下这样：

![First notification without merging.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/desktop/merge-notification-first.png)

第二条通知会以这样的方式覆盖已有的通知：

![Second notification with merging.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/desktop/merge-notification-second.png)

这种方法的好处是，如果你的用户目睹了通知一个接着一个的出现，相比于只用最新的信息替代当前的通知内容，消息看起来则更紧密相连。

## 各种规则的例外

我一直强调的是，你**必须**在收到推送信息时展示通知，这个规则在**大多数**情况下是正确的。只有在一种场景下你不需要展示通知，
那就是用户已经打开了你的站点，并且站点已经是呼起的状态。

在你的推送事件中，你可以通过检测目标窗口是否已被打开并呼起，来决定是否需要展示通知。

获得浏览器所有窗口、查询当前已呼起窗口的代码可以参考如下：

    function isClientFocused() {
      return clients.matchAll({
        type: 'window',
        includeUncontrolled: true
      })
      .then((windowClients) => {
        let clientIsFocused = false;

        for (let i = 0; i < windowClients.length; i++) {
          const windowClient = windowClients[i];
          if (windowClient.focused) {
            clientIsFocused = true;
            break;
          }
        }

        return clientIsFocused;
      });
    }

我们一般使用[`clients.matchAll()`](https://developer.mozilla.org/en-US/docs/Web/API/Clients/matchAll)来获得当前浏览器下所有窗口对象，
然后遍历其结果去检查`focused`参数。

在推送事件内，我们会使用如下方法来决定是否展示通知：

      const promiseChain = isClientFocused()
      .then((clientIsFocused) => {
        if (clientIsFocused) {
          console.log('Don\'t need to show a notification.');
          return;

        }

        // 窗口并没有被呼起，我们需要展示通知
        return self.registration.showNotification('Had to show a notification.');
      });

      event.waitUntil(promiseChain);

## 通过推送事件给页面发送消息

我们已知，可以在用户正在浏览我们站点的时候不进行通知。但是，如果你仍然想让用户知道这个推送事件已经发生，但又觉得进行通知太过严肃，应该如何处理？

其中一个方法就是利用service worker给页面发送信息，页面可以给用户展示通知或更新，让用户知晓到这个推送事件。当然，只有当用户对于不易察觉的通知感到更友好时，这种做法才有用。

如果我们接收到了一个推送，并且检测到了我们的APP已经被打开了，那么我们就可以"发送消息"给每个打开的页面，就像以下这样：

      const promiseChain = isClientFocused()
      .then((clientIsFocused) => {
        if (clientIsFocused) {
          windowClients.forEach((windowClient) => {
            windowClient.postMessage({
              message: 'Received a push message.',
              time: new Date().toString()
            });
          });
        } else {
          return self.registration.showNotification('No focused windows', {
            body: 'Had to show a notification instead of messaging each page.'
          });
        }
      });

      event.waitUntil(promiseChain);

在每个页面中，我们通过监听消息（message）事件来接收消息：

        navigator.serviceWorker.addEventListener('message', function(event) {
          console.log('Received a message from service worker: ', event.data);
        });

在这个消息监听器中，你可以做任何你想做的事，例如展示自定义的视图，或者完全忽略这个消息。

值得注意的是，如果你没有在你的网页中定义消息监听器，那么service worker推送的消息将不会做任何事。


## 缓存页面和窗口对象

有一个场景可能超出了这本书的范畴，但是依然值得探讨，那就是缓存你希望用户点击通知后访问的网页，以此来提升web应用的整体用户体验。

这就需要设置你的service worker来处理这些`fetch`事件，但如果你监听了`fetch`事件，请确保在展示你的通知之前，通过缓存你需要的页面和资源来充分利用它在`push`事件中的优势。

想要了解更多缓存相关信息，请参考[introduction to service workers post](/web/fundamentals/getting-started/primers/service-workers)[introduction to service workers post](/web/fundamentals/getting-started/primers/service-workers]
