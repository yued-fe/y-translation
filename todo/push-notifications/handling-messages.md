>原文地址：https://developers.google.com/web/fundamentals/push-notifications/handling-messages

>译文地址：

>译者：张卓

>校对者：


# 推送事件 





到此已经覆盖了订阅用户并给其推送发送消息。下一步是在用户的设备上接收此推送消息并显示通知（以及我们可能要做的任何其他工作）。

## 推送事件

当接受到一条消息时，一个推送事件将被 dispatch 到你的 service worker 中。

监听推送事件的代码和在 JavaScript 中监听其它事件的代码十分类似：

    self.addEventListener('push', function(event) {
      if (event.data) {
        console.log('This push event has data: ', event.data.text());
      } else {
        console.log('This push event has no data.');
      }
    });

对于不熟悉 service workers 的开发者来说，这段代码最奇怪的部分应该是变量 `self`。`self` 常用在 Web Workers 中， service workers 就是这样的。`self` 是指全局作用域，类似于浏览器环境中的 `window`。但对于 web workers 和 service workers，`self` 指的的 worker 本身。

在上面的例子中，可以将 `self.addEventListener()` 视为向 service workers 本身添加事件监听。

在推送事件示例中，我们检查是否有数据，并打印一些日志到终端。

除此之外，还有其他方法可以解析推送事件中的数据：

    // 返回 string
    event.data.text()

    // Parses data as JSON string and returns an Object
    event.data.json()

    // 返回 blob
    event.data.blob()

    // 返回 arrayBuffer
    event.data.arrayBuffer()

大多数人使用 `json()` 或者 `text()`（取决于他们希望从应用程序那得到什么）。

此示例演示如何添加推送事件监听器以及如何访问数据，但是它
缺少两个非常重要的功能：它没有显示通知，并且没有使用 `event.waitUntil()`。

### Wait Until

必须要了解的一点是，你几乎无法控制 service workers 的代码何时运行，浏览器决定何时将其唤醒以及何时终止它。唯一的方法是，你告诉浏览器：“嘿——我非常忙，要做重要的事情了”，将一个 promise 对象传递给 `event.waitUntil()` 方法，这样，浏览器就会
保持 service workers 运行，直到传入的 promise 被 resolve。

对于推送事件，还有一个必要条件是必须在 promise 被 resolve 之前显示通知。

以下是显示通知的基本示例：

    self.addEventListener('push', function(event) {
      const promiseChain = self.registration.showNotification('Hello, World.');

      event.waitUntil(promiseChain);
    });

执行 `self.registration.showNotification()` 方法会向用户显示一个通知并且返回一个 promise 对象，这个 promise 对象会在通知显示之后被 resolve。

为了让这个例子尽可能清楚，我已经将这个 promise 对象赋值给了一个
叫做 `promiseChain` 的变量，然后将其传递给 `event.waitUntil()`。 我知道这里
非常冗长，但我已经看到了许多由此引发的问题，例如，
误解了应该传递给 `waitUntil()` 的内容，或者是一个错误的 promise 链。

一个包括网络请求数据和分析追踪推送事件的例子如下：

    self.addEventListener('push', function(event) {
      const analyticsPromise = pushReceivedTracking();
      const pushInfoPromise = fetch('/api/get-more-data')
        .then(function(response) {
          return response.json();
        })
        .then(function(response) {
          const title = response.data.userName + ' says...';
          const message = response.data.message;

          return self.registration.showNotification(title, {
            body: message
          });
        });

      const promiseChain = Promise.all([
        analyticsPromise,
        pushInfoPromise
      ]);

      event.waitUntil(promiseChain);
    });


这里为了示例，我们调用一个返回 promise 对象的函数 `pushReceivedTracking()`，
，假装将发出网络请求到我们的分析提供商。同时，我们也会发送网络请求、获取响应，并使用响应的数据来显示通知的标题和内容。

我们使用 `Promise.all()` 将这两个 promise 对象合并，来确保 service worker 在这两个任务完成之前存活。合并后的 promise 被传递进   `event.waitUntil()` ，这意味着浏览器将等到两个 promise 都完成后，才会检查通知已显示，最后终止 service worker。

注意：如果你对 promise 链式调用有些疑惑，将功能分解为函数会有助于降低复杂性，同时也推荐这篇[Philip Walton 的博文](https://philipwalton.com/articles/untangling-deeply-nested-promise-chains/)来理解 promise 的链式调用。重点是你应该尝试如何来写 promise 以及它的链式调用，最终找到适合自己的风格。

我们应该关注 `waitUntil()` 以及如何使用它，因为开发人员常常会面临的一个问题是，当 promise 链使用的不正确时，Chrome 会
显示此“默认”通知：

![An Image of the default notification in Chrome](./images/default-notification-mobile.png)

当接收到一个推送消息，但在 service worker 中的推送事件当传递给 `event.waitUntil()` 的 promise 结束之后也没有显示任何消息，Chrome 就只会显示 "This site has been updated in the background." 

导致这个问题主要原因是开发者的代码中经常在调用 `self.registration.showNotification()` 之后在 promise 中 **没有**返回任何东西，这会导致显示默认通知。举个例子，我们可以删除上面示例中的 `self.registration.showNotification()` 的返回值，就会有看到“默认”通知的风险。

    self.addEventListener('push', function(event) {
      const analyticsPromise = pushReceivedTracking();
      const pushInfoPromise = fetch('/api/get-more-data')
        .then(function(response) {
          return response.json();
        })
        .then(function(response) {
          const title = response.data.userName + ' says...';
          const message = response.data.message;

          self.registration.showNotification(title, {
            body: message
          });
        });

      const promiseChain = Promise.all([
        analyticsPromise,
        pushInfoPromise
      ]);

      event.waitUntil(promiseChain);
    });

你可以看到它是如何容易遗漏的。

请记住 - 如果看到该通知，请检查 promise 链和 `event.waitUntil()`。

在下一节中，我们将看看我们可以做什么来设置通知的样式以及可以展示什么内容。
