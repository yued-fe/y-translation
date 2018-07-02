>原文地址：https://developers.google.com/web/fundamentals/push-notifications/notification-behaviour

>译文地址：

>译者：任家乐

>校对者：

# Notification Behaviour
通知行为


So far we've looked at the options that alter the visual appearance of a notification. There
are also options that alter the behaviour of notifications.
到此为止，我们已经浏览了可以改变通知样式的选项，除了样式，我们还可以通过一些选项来改变通知的行为。

Be default, calling `showNotification()` with just visual options will have
the following behaviours:
默认情况下，在设置好视觉展示相关选项时，调用"showNotification()" 会出现以下行为：

- Clicking on the notification does nothing.
- 点击通知不会触发任何事件。

- Each new notification is shown one after the other. The browser will not collapse the
notifications in any way.
- 每个新的通知会逐一有序的展示，浏览器不会以任何方式叠加展示通知。

- The platform may play a sound or vibrate the user's devices (depending on the platform).
- 系统会以音效或震动设备的方式提示用户（具体方式则取决于设备系统）。

- On some platforms the notification will disappear after a short
period of time while others will show the notification unless the user interacts with it.
(For example, compare your notifications on Android and Desktop.)
- 在一些系统上，短时间后通知会自动消失，而其他的系统则会一直展示通知直到用户对其进行操作。（可以对比安卓和电脑的通知行为）

In this section we are going to look at how we can alter these default behaviours using options
alone. These are relatively easy to implement and take advantage of.
在这一节中，我们会探讨下单独使用这些选项会如何改变默认的通知行为，这些都相对来说比较容易实施并且都具有优势。

### Notification Click Event
### 通知的点击事件

When a user clicks on a notification the default behaviour is for nothing
to happen. It doesn't even close or remove the notification.
当用户点击通知时，默认不会触发任何事件，它并不会关闭或移除通知。

The common practice for a notification click is for it to close and perform some other logic
(i.e. open a window or make some API call to the application).
通知点击事件的常见用法是调用它来关闭并展示一些其他的交互逻辑（例如，打开一个窗口或调用一些可用的API）

To achieve this we need to add a 'notificationclick' event listener to our service worker. This
will be called when ever a notification is clicked.
要实现以上所说的，我们需要让service worker来监听'notificationclick'事件。这个事件在每次点击通知时都被调用。

    self.addEventListener('notificationclick', function(event) {
      const clickedNotification = event.notification;
      clickedNotification.close();

      // Do something as the result of the notification click
      const promiseChain = doSomething();
      event.waitUntil(promiseChain);
    });

As you can see in this example, the notification that was clicked can be
accessed via the `event.notification` parameter. From this we can access
the notifications properties and methods. In this case we call its
`close()` method and perform additional work.

从以上demo代码可以知道，被点击的通知可以通过event.notification参数来访问。通过这个参数我们可以获得通知的属性和方法，因此我们能够调用通知的'close()'方法
，同时执行一些额外的操作。


Note: You still need to make use of event.waitUntil() to keep the service worker running
while your code is busy.
提示：在程序运行高峰期，你仍然需要利用event.waitUntil()方法保证service worker的持续运行。

### Actions
### 行为

Actions allow you to give users another level of interaction with your users
over just clicking the notification.
行为的使用可以提供给用户另一个水准的交互体验，相比于之前的普通点击行为。

In the previous section you saw how to define actions when calling
`showNotification()`:
在上一节中，我们get到了如何调用showNotification()来定义行为：

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

If the user clicks an action button, check the `event.action` value in
the `noticationclick` event to tell which action button was clicked.
如果用户点击了可以触发行为的按钮，通过'notificationclick'回调中返回的'event.action'就可以知道被点击的按钮是哪个。

`event.action` will contain the `action` value set in the options. In the example above the
`event.action` values would be one of the following: 'coffee-action', 'doughnut-action',
'gramophone-action' or 'atom-action'.
'event.action'

'event.action'会包含所有选项中有关行为（action）的值的集合。在上面的例子中，'event.action'的值则会是'coffee-action', 'doughnut-action',
'gramophone-action' 或 'atom-action'的其中一个。

With this we would detect notification clicks or action clicks like so:
因此通过event.action，我们可以依照以下代码来监听通知或行为（action）的点击事件。

    self.addEventListener('notificationclick', function(event) {
      if (!event.action) {
        // Was a normal notification click
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

![Logs for action button clicks and notification click.](./images/notification-screenshots/action-button-click-logs.png)

### Tag
### 标签

The *tag* option is essentially a string ID that "groups" notifications together, providing
an easy way to determine how multiple notifications are displayed to the user. This is easiest
to explain with an example.
标签选项本质上可以理解为：使用一个字符串类型的ID将通知"聚集"到一起，同时提供了一个简单的方式来抉择多个通知的展示方式，这里可能用示例来解释最为简单：

Let's display a notification and give it a tag, of
'message-group-1'. We'd display the notification with this code:
让我们来展示一个通知，并给它标记一个标签，例如'message-group-1'. 我们可以按照以下代码来展示这个通知：

        const title = 'Notification 1 of 3';
        const options = {
          body: 'With \'tag\' of \'message-group-1\'',
          tag: 'message-group-1'
        };
        registration.showNotification(title, options);

Which will show our first notification.
以上代码会展示我们定义好的第一个通知。

![First notification with tag of message group 1.](./images/notification-screenshots/desktop/chrome-first-tag.png)

Let's display a second notification with a new tag of 'message-group-2', like so:
我们再用一个新的标签'message-group-2'来标记并展示第二个通知，例如以下：

            const title = 'Notification 2 of 3';
            const options = {
              body: 'With \'tag\' of \'message-group-2\'',
              tag: 'message-group-2'
            };
            registration.showNotification(title, options);

 This will display a second notification to the user.
 以上代码会展示给用户第二个通知。

![Two notifications where the second tag is message group 2.](./images/notification-screenshots/desktop/chrome-second-tag.png)

Now let's show a third notification but re-use the first tag of 'message-group-1'. Doing this
will close the first notification and replace it with our new notification.
现在让我们展示第三个通知，但不新增标签，而是重用我们第一次定义的标签'message-group-1'。这么操作会关闭之前的第一个通知并将其替换成新定义的通知。

            const title = 'Notification 3 of 3';
            const options = {
              body: 'With \'tag\' of \'message-group-1\'',
              tag: 'message-group-1'
            };
            registration.showNotification(title, options);

Now we have two notifications even though `showNotification()` was called three times.
现在即使我们3次调用'showNotification()'也只会展示2个通知。

![Two notifications where the first notification is replaced by a third notification.](./images/notification-screenshots/desktop/chrome-third-tag.png)
The `tag` option is simply a way of grouping messages so that any old notifications that
are currently displayed will be closed if they have the same tag as a new notification.
标签这个选项简单来看就是一个用于分类信息的方式，因此在新通知与已有通知标记为同一个标签时，当前被展示的所有旧通知将会被关闭。

A subtlety to using `tag` is that when it replaces a notification, it
will do so *without* a sound and vibration.
使用标签有一个需要注意的小细节：当它替换了一个通知时，是没有音效和震动提醒的。

This is where the `renotify` option comes in.
此时'Renotify'选项就有了用武之地。

### Renotify
### 是否替换之前的通知

This largely applies to mobile devices at the time of writing. Setting this option makes new
notifications vibrate and play a system sound.
这个选项大多数情况应用于写作时间。通过设置它，接收到新的通知时，系统会震动并播放系统音效。

There are scenarios where you might want a replacing notification to
notify the user rather than silently update. Chat applications are a good
example. In this case you should set `tag` and `renotify` to true.
某些场景下，你可能更希望替换通知时能够提醒到用户，而不是默默地进行。聊天应用则是一个很好的例子。这种情况你需要同时使用'标签'和'是否替换之前的通知(Renotify)'选项。

            const title = 'Notification 2 of 2';
            const options = {
              tag: 'renotify',
              renotify: true
            };
            registration.showNotification(title, options);

**Note:** If you set `renotify: true` on a notification without a tag, you'll get the following
error:

    TypeError: Failed to execute 'showNotification' on 'ServiceWorkerRegistration':
    Notifications which set the renotify flag must specify a non-empty tag

**注意：** 如果你设置了'是否替换之前的通知'(Renotify)为true但却没有设置标签，会出现以下报错信息：
    类型错误：不能够在'ServiceWorkerRegistration'上执行'showNotification'方法：设置了renotify标识的通知必须声明一个不为空的标签。

### Silent
### 静音

This option allows you to show a new notification but prevents the default
behavior of vibration, sound and turning on the device's display.
这一选项可以阻止默认的震动及音效来展示新的提示，同时会打开设备的屏幕。

This is ideal if your notifications don't require immediate attention
from the user.
如果你的通知不需要立马让用户注意到，这个选项就是最合适的方法。

        const title = 'Silent Notification';
        const options = {
          silent: true
        };
        registration.showNotification(title, options);

**Note:** If you define both *silent* and *renotify*, silent will take precedence.
**注意** 如果同时设置了*silen(静音)*和*renotify(重复通知)*，silent(静音)选项会取得更高的优先级。

### Requires Interaction
### 是否需要与通知进行交互

Chrome on desktop will show notifications for a set time period before hiding them. Chrome on
Android doesn't have this behaviour. Notifications are displayed until
the user interacts with them.
桌面的chrome浏览器会在通知消失前展示一段时间。而安卓设备的chrome浏览器不会有这种行为，通知会一直展示，直到用户对其进行操作。

To force a notification to stay visible until the user interacts with it
add the `requireInteraction` option. This will show the notification
until the user dismisses or clicks your notification.
如果要强制让通知持续展示直到用户对其操作，需要添加'requireInteraction'选项，这个选项会展示通知直到用户划走（驳回）或点击它。

        const title = 'Require Interaction Notification';
        const options = {
          body: 'With "requireInteraction: \'true\'".',
          requireInteraction: true
        };
        registration.showNotification(title, options);

Please use this option with consideration. Showing a notification and forcing the user to stop
what they are doing to dismiss your notification can be frustrating.
请谨慎使用这个选项，一直展示通知、并强制让用户停下手头的事情来取消通知可能会干扰到用户。

In the next section we are going to look at some of the common patterns used on the web for
managing notifications and performing actions such as opening pages when a notification is clicked.
在下一节中，我们会看一些可以在web上适用的常见模式来管理通知，同时在点击通知时执行打开网页的行为。

