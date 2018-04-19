
- 原文地址：https://blog.bloomca.me/2018/04/17/asynchronous-javascript-pattern-promises-tips-and-tricks.html
- 译者：[名称](git、知乎、掘金、个人网站或者微博的链接)
- 校对者：[名称](git、知乎、掘金、个人网站或者微博的链接)

Promises are to this point a very popular concept in JavaScript – they are native across all modern browsers, Node is switching to promises from callback style (there is even a way to convert [callback style to promise style](https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original)), but there are some small things which are easy to overlook.

## Promisifying sync values

Sometimes we receive a value, but we are not sure whether it is promise or not. In this case, we can use [Promise.resolve](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve), which wraps the value into a promise, but if the value itself is a promise, it will wait until its resolution, so it is totally safe:

```
Promise.resolve(syncValue).then(sameSyncValue => ...);
Promise.resolve(promiseValue).then(resolvedPromiseValue => ...);

```

If the value is sync, handler will be executed as a microtask (read more in [Jake Archibald’s article](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)), so it should be executed right after current sync expressions, but before next tick.

This technique is also handy in case you have a function which might return sync values, but in some cases has to execute some async code – you can just switch to promise interface, wrapping sync values in `Promise.resolve()`.

> if we use [async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and don’t use any `await` inside, and just return sync value, it works like it was wrapped in `Promise.resolve()`

## Changing data from promise

Sometimes we want to change data coming from an async action – e.g. we want to add one more object before returning the promise. We can do that easily using the fact that in the promise chain functions we can safely return sync values, so we need just to write a custom `then` handler:

```
return getUserSubscriptions(user).then(subscriptions => {
  // change resolved value – add user object as well
  return {
    subscriptions,
    user
  };
});

```

So, instead of return original promise, we handle success callback by modifying resolved promise value, and return it from our function. Also, you can use this technique just to modify data from your promise (without adding new parts) – for example, if you use [adapter pattern](https://en.wikipedia.org/wiki/Adapter_pattern) to match data which your application expects:

```
// we modify data from the server to serve our application better:
return getUserData().then(parseUserData);

```

> there is also a related article about [passing data between promise callbacks](http://2ality.com/2017/08/promise-callback-data-flow.html) from Dr. Axel Rauschmayer

## Promise.race caveat

[Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) allows us to create a promise, which will be resolved (or rejected) as soon as only *one* promise from the array is resolved (or rejected).

There is one caveat, which totally makes sense, but easy to overlook: it we pass an empty array into `Promise.race([])`, it will never be resolved. This is intended, and specificed [in the specification](https://www.ecma-international.org/ecma-262/6.0/#sec-promise.race) (see Note 1), so if you have a dynamically constructed array for `Promise.race`, wrap it in a length check condition:

```
if (promises.length === 0) {
  return Promise.resolve();
} else {
  return Promise.race(promises);
}

```

## Promises with timeout

One possible usage of `Promise.race()` is to set a timeout for our promise – so, let’s say we want to tell user that everything is good anyway after 5 seconds. In order to do so, we’ll need to wait for the first resolved promise – it is not that important for us, what will be resolved first, but we definitely don’t want to wait 5 seconds all the time:

```
// simple implementation of `wait`, `delay`, `sleep` etc
// function which resolves after passed time in ms
function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// our logic:
const promise = makeAsyncAction();
// our promise will be resolved either after our async action
// or, if it takes more than 5 seconds, after these 5 seconds
const limitedFor5SecondsPromise = Promise.race([
  promise,
  wait(5000)
]);

// you might want to return both promises, in case async one fails
// or just to show background loader
return { promise, limitedFor5SecondsPromise };

```

As I mentioned in the commentary before returning, you still might want to have original promise reference: you can handle a failure, and show some background loader indicating that something is still in the progress, but it does not block the UI.

## 2nd argument in `then`

[Promise.prototype.then](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) accepts two parameters – first one is a success handler, and second one is for a failure. While it resembles `catch` a lot, they are in fact different.

Let’s put an example first:

```
// first way
someAsyncCode()
  .then(successHandler)
  .catch(errorHandler);

// another approach
someAsyncCode()
  .then(successHandler, errorHandler);

```

First things first – `catch` is nothing else but [then(undefined, errorHandler)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch). With this in mind, we can rewrite our first example as:

```
someAsyncCode()
  .then(successHandler)
  .then(undefined, errorHandler);

```

So, what is the difference? The difference is in *what* exactly is handled by our `errorHandler` – it handles *previous* promise in the chain: in the first case, it is `someAsyncCode().then(successHandler)`, and in the second one, it is only `someAsyncCode()`. So, first case handles both original call and `successHandler`, but the second one, in case of throwing or returning rejected promise in `successHandler`, won’t handle it.

This is not what you usually expect – normally you just want to handle all promises before previous `catch` handler, so to avoid confusion Bluebird [recommends to avoid it](https://github.com/petkaantonov/bluebird/wiki/Promise-anti-patterns#the-thensuccess-fail-anti-pattern).

## Execution order

This is not a caveat really, but promises make it very easy to chain async actions, even if they can be done in parallel:

```
return getUser(userId)
  .then(user => {
    return getSubscriptions(userId)
      .then(subscriptions => ({ subscriptions, user }))
  })
  .then(data => {
    return getFavourites(userId)
      .then(favourites => ({ ...data, favourites }));
  });

```

In this example we had to pass data around, so it is not as elegant, but imagine, we want to make several `POST` requests, and don’t want necessarily use result from them:

```
return updateUser(user)
  .then(() => updateMarketing(marketing))
  .then(() => updateSubscriptions(subscriptions));

```

In this example we don’t depend on results from the previous call, and single thing which is important for us is when all calls were finished (or if any failed). Moreover, it has problems – in case any call fails, following functions won’t even execute – even though we might be totally fine updating subscriptions in case marketing updates failed.

You can use [Promise.all](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) to make all these requests in parallel (it will return a new promise, which will be resolved as soon as all promises are resolved, with an array of resolved values):

```
// first example
return Promise.all([
  getUser(userId),
  getSubscriptions(userId),
  getFavourites(userId)
]).then(([user, subscriptions, favourites]) => ({
  user, subscriptions, favourites
}));

// second example
return Promise.all([
  updateUser(user),
  updateMarketing(marketing),
  updateSubscriptions(subscriptions)
]);

```

Now all requests will be done in parallel, bringing you speed. It also shows that these requests don’t depend on each other, and won’t create confusion in the future, in case of refactoring. These are fairly simple examples, but it is totally fine to have several parallel promise “tracks”, which will be merged in the end – in case of big number of requests and so fast network, it might improve response time significantly.
