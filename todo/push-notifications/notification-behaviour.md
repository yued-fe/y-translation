>原文地址：https://developers.google.com/web/fundamentals/push-notifications/notification-behaviour

>译文地址：https://github.com/yued-fe/y-translation/edit/master/todo/push-notifications/notification-behaviour.md

>译者：[任家乐](https://github.com/jennyrenjiale)

>校对者：[刘文涛](https://github.com/HSDPA-wen)、[杨芯芯](https://github.com/y2x33)

# 通知行为

到此为止，我们已经浏览了可以改变通知样式的选项，除了样式，我们还可以通过一些选项来改变通知的行为。
默认情况下，如果只设置视觉相关选项时，调用`showNotification()`会出现以下行为：

- 点击通知不会触发任何事件。
- 每个新的通知会逐一有序地展示，浏览器不会以任何方式叠加展示通知。
- 系统会以音效或震动的方式提示用户（具体方式则取决于设备系统）。
- 在某些系统上，通知会在短时间展示后消失，而其他系统则会一直展示通知直到用户对其进行操作。（可以对比安卓和电脑的通知行为）

在这一节中，我们会探讨如何单独使用一些选项改变默认的通知行为，这相对来说比较容易实施和利用。

### 通知的点击事件

当用户点击通知时，默认不会触发任何事件，它并不会关闭或移除通知。

通知点击事件的常见用法是调用它来关闭通知、同时执行一些其他的逻辑（例如，打开一个窗口或对应用程序进行一些API调用）

为此，我们需要在service worker中添加一个 “ notificationclick ” 事件监听器。 这个事件将在点击通知时被调用。

    self.addEventListener('notificationclick', function(event) {
      const clickedNotification = event.notification;
      clickedNotification.close();

      // 点击通知后做些什么
      const promiseChain = doSomething();
      event.waitUntil(promiseChain);
    });

正如你在此示例中所看到的，被点击的通知可以通过`event.notification`参数来访问。通过这个参数我们可以获得通知的属性和方法，因此我们能够调用通知的`close()`方法，同时执行一些额外的操作。

提示：在程序运行高峰期，你仍然需要利用event.waitUntil()方法保证service worker的持续运行。

### Actions

相比于之前的普通点击行为，`actions`的使用可以提供给用户更高级别的交互体验。

在上一节中，我们知道了如何调用`showNotification()`来定义`actions`：

        const title = 'Actions Notification';
        const options = {
          actions: [
            {
              action: 'coffee-action',
              title: 'Coffee',
              icon: '/images/demos/action-1-128x128.png'
            },
            {
              action: 'doughnut-action',
              title: 'Doughnut',
              icon: '/images/demos/action-2-128x128.png'
            },
            {
              action: 'gramophone-action',
              title: 'gramophone',
              icon: '/images/demos/action-3-128x128.png'
            },
            {
              action: 'atom-action',
              title: 'Atom',
              icon: '/images/demos/action-4-128x128.png'
            }
          ]
        };

        const maxVisibleActions = Notification.maxActions;
        if (maxVisibleActions < 4) {
          options.body = `This notification will only display ` +
            `${maxVisibleActions} actions.`;
        } else {
          options.body = `This notification can display up to ` +
            `${maxVisibleActions} actions.`;
        }

        registration.showNotification(title, options);

如果用户点击了action按钮，通过`notificationclick`回调中返回的`event.action`就可以知道被点击的按钮是哪个。

'event.action'会包含所有选项中有关`action`的值的集合。在上面的例子中，`event.action`的值则会是： “coffee-action”、 “doughnut-action”,
“gramophone-action” 或 “atom-action” 的其中一个。

因此通过event.action，我们可以检测到通知或action的点击，代码如下：

    self.addEventListener('notificationclick', function(event) {
      if (!event.action) {
        // 正常的通知点击事件
        console.log('Notification Click.');
        return;
      }

      switch (event.action) {
        case 'coffee-action':
          console.log('User ❤️️\'s coffee.');
          break;
        case 'doughnut-action':
          console.log('User ❤️️\'s doughnuts.');
          break;
        case 'gramophone-action':
          console.log('User ❤️️\'s music.');
          break;
        case 'atom-action':
          console.log('User ❤️️\'s science.');
          break;
        default:
          console.log(`Unknown action clicked: '${event.action}'`);
          break;
      }
    });

![Logs for action button clicks and notification click.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/action-button-click-logs.png)

### Tag（标签）

**tag** 选项的本质是一个字符串类型的ID，以此将通知 “分组” 在一起，并提供了一种简单的方法来向用户显示多个通知，这里可能用示例来解释最为简单：

让我们来展示一个通知，并给它标记一个标签，例如“message-group-1”. 我们可以按照以下代码来展示这个通知：

        const title = 'Notification 1 of 3';
        const options = {
          body: 'With \'tag\' of \'message-group-1\'',
          tag: 'message-group-1'
        };
        registration.showNotification(title, options);

以上代码会展示我们定义好的第一个通知。

![First notification with tag of message group 1.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/desktop/chrome-first-tag.png)

我们再用一个新的标签 “message-group-2” 来标记并展示第二个通知，如下：

            const title = 'Notification 2 of 3';
            const options = {
              body: 'With \'tag\' of \'message-group-2\'',
              tag: 'message-group-2'
            };
            registration.showNotification(title, options);

 以上代码会展示给用户第二个通知。

![Two notifications where the second tag is message group 2.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/desktop/chrome-second-tag.png)

现在让我们展示第三个通知，但不新增标签，而是重用我们第一次定义的标签 “message-group-1”。这么操作会关闭之前的第一个通知并将其替换成新定义的通知。

            const title = 'Notification 3 of 3';
            const options = {
              body: 'With \'tag\' of \'message-group-1\'',
              tag: 'message-group-1'
            };
            registration.showNotification(title, options);

现在即使我们连续3次调用`showNotification()`也只会展示2个通知。

![Two notifications where the first notification is replaced by a third notification.](https://developers.google.com/web/fundamentals/push-notifications/images/notification-screenshots/desktop/chrome-third-tag.png)

`tag`这个选项简单来看就是一个用于信息分组的方式，因此在新通知与已有通知标记为同一个标签时，当前被展示的所有旧通知将会被关闭。

使用`tag`有一个容易被忽略的小细节：当它替换了一个通知时，是**没有**音效和震动提醒的。

此时`Renotify`选项就有了用武之地。

### Renotify（是否替换之前的通知）

在写此文时，这个选项大多数应用于移动设备。通过设置它，接收到新的通知时，系统会震动并播放系统音效。

某些场景下，你可能更希望替换通知时能够提醒到用户，而不是默默地进行。聊天应用则是一个很好的例子。这种情况你需要同时使用`tag`和`Renotify`选项。

            const title = 'Notification 2 of 2';
            const options = {
              tag: 'renotify',
              renotify: true
            };
            registration.showNotification(title, options);

    TypeError: Failed to execute 'showNotification' on 'ServiceWorkerRegistration':
    Notifications which set the renotify flag must specify a non-empty tag

**注意：** 如果你设置了`Renotify: true`但却没有设置标签，会出现以下报错信息：

类型错误：不能够在 “ServiceWorkerRegistration” 上执行 “showNotification” 方法：设置了renotify标识的通知必须声明一个不为空的标签。(TypeError: Failed to execute 'showNotification' on 'ServiceWorkerRegistration':Notifications which set the renotify flag must specify a non-empty tag)

### Silent（静音）

这一选项可以阻止设备震动、音效以及屏幕亮起的默认行为。如果你的通知不需要立马让用户注意到，这个选项是最合适的。

        const title = 'Silent Notification';
        const options = {
          silent: true
        };
        registration.showNotification(title, options);

**注意：** 如果同时设置了**silent**和**Renotify**，silent选项会取得更高的优先级。

### 与通知进行交互

桌面chrome浏览器会展示通知一段时间后将其隐藏，而安卓设备的chrome浏览器不会有这种行为，通知会一直展示，直到用户对其进行操作。

如果要强制让通知持续展示直到用户对其操作，需要添加`requireInteraction`选项，此选项会展示通知直到用户消除或点击它。

        const title = 'Require Interaction Notification';
        const options = {
          body: 'With "requireInteraction: \'true\'".',
          requireInteraction: true
        };
        registration.showNotification(title, options);

请谨慎使用这个选项，因为一直展示通知、并强制让用户停下手头的事情来取消通知可能会干扰到用户。

在下一节中，我们会浏览一些web上适用的用于管理通知的常见模式，例如在点击通知时执行打开网页的行为。

