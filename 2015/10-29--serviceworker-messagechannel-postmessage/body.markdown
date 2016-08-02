# Messaging the ServiceWorker

In order to message a `ServiceWorker`, you can use a piece of code like the following snippet.

```js
worker.postMessage(data);
```

By default you probably want to use `navigator.serviceWorker.controller`, the active ServiceWorker instance for the current page that controls its requests, as the `worker`. I haven't *personally* found a need yet for talking to workers other than the controller, _which is why that's the default `worker` you talk to in [`swivel`][1]_. That being said, there are use cases for them, which is why there's [`swivel.at(worker)`][2], but let's go back to our focus area.

The worker can set up an event handler that deals with the `data` payload we posted from the web page.

```js
self.addEventListener('message', function handler (event) {
  console.log(event.data);
});
```

As soon as you have different types of messages you want to send to workers, this becomes an issue. You'll have to turn to a convention for message routing. A common convention is to define an envelope for your messages that has a `command` property in it, so now you'd send messages like the following.

```js
worker.postMessage({ <mark>command: 'deleteCache'</mark>, key: key });
```

Then the worker needs to be updated with a command handling router, a bunch of `if` will do in the simpler case. You can see how the code starts becoming diluted with implementation details.

```js
self.addEventListener('message', function handler (event) {
  if (<mark>event.data.command === 'deleteCache'</mark>) {
    caches.delete(event.data.key);
  }
});
```

# Getting Replies from the `ServiceWorker`

If you want the worker to be able to reply to the message **things get very ugly, very quickly**, mostly because browsers haven't implemented the final API quite yet.  
For the time being, on the client-side you'll need to set up a `MessageChannel`, bind a listener to `port1` and pass along `port2` when posting the message.

```js
var messageChannel = <mark>new MessageChannel()</mark>;
messageChannel.<mark>port1</mark>.addEventListener('message', replyHandler);
worker.postMessage(data, <mark>[messageChannel.port2]</mark>);
function replyHandler (event) {
  console.log(event.data); // <mark>this comes from the ServiceWorker</mark>
}
```

On the `ServiceWorker` side, it's not that fun either. You have to reference `port2` of the `messageChannel` using `event.ports[0]`, as its the port in position zero of the `ports` passed along with the message.

```js
self.addEventListener('message', function handler (event) {
  <mark>event.ports[0].postMessage(data)</mark>; // handle this using the <mark>replyHandler</mark> shown earlier
});
```

Browsers will eventually have an `event.source` alternative to `event.ports[0]` on the `ServiceWorker` side that doesn't need us to do any of the `MessageChannel` stuff on the pages. Unfortunately that's _not here yet_, and so we have to resort to `MessageChannel` for now.

# Broadcasting from a ServiceWorker to every client

This one is straightforward, but it's also pretty different from the two situations we've just talked about. And, quite honestly, _very_ verbose. Of course, `ServiceWorker` is all about expressiveness and being able to cater for multiple different use cases, and we've all seen the veiled evil in [seemingly simple but cleverly complicated][3] interfaces like the AppCache manifest.

That being said, having to type this out sucks. Libraries will definitely help abstract the pain away, while `ServiceWorker` can go on being just about [the most powerful feature][4] the modern web has to offer.

```js
self.<mark>clients.matchAll()</mark>.then(all => all.map(client => <mark>client.postMessage</mark>(data)));
```

To listen to these messages from a `ServiceWorker`, you can register an event listener like below in your web page. Note how the listener is added on `navigator.serviceWorker` and not on an specific `worker` _(like `navigator.serviceWorker.controller`)_.

```js
navigator.serviceWorker.addEventListener('message', function handler (event) {
  console.log(event.data);
});
```

You probably want to keep a reference to the `worker` you're interested in, and then filter on the message listener by `event.source`. That way you'll avoid messages broadcasted from workers other than the one you're expecting messages from.

```js
navigator.serviceWorker.addEventListener('message', function handler (event) {
  if (event.source !== <mark>worker</mark>) {
    return;
  }
  console.log(event.data);
});
```

# Dual Channeling `fetch` Requests

Sending updates to the origin of `fetch` requests is, perhaps, the most interesting use case for communication between `ServiceWorker` and web pages. Sending an update to the origin of a `fetch` request is what makes using the cache immediately and then sending an update as soon as possible so effective.

```js
self.on('fetch', function handler (event) {
  // reply to client here
});
```

Eventually, `event` will have a `clientId` property identifying the client where the request came from. We could then use code like below to send a message to the `client` using `client.postMessage`. No browser implements `event.clientId` yet.

```js
self.on('fetch', function handler (event) {
  event.respondWith(caches.match(event.request));
  fetch(event.request).then(response => response.json()).then(function (data) {
    self.clients.match(<mark>event.clientId</mark>).then(client => client.postMessage(data));
  });
});
```

You could still use the broadcasting mechanism discussed earlier, but it'd go to every client and not just the origin of the `fetch` event. A better alternative, for now, may be to issue another request from the page after load, maybe adding a custom header indicating that we should force a `fetch` on the `ServiceWorker` side.

# Swivel Makes Your Life Easier

If you'd prefer to avoid all of this frivolous knowledge, you may like to [`swivel`][1] like Ron Swanson.

![Ron Swanson swivelling to avoid human contact][5]

Swivelling has a number of benefits. The API is unified under an event emitter style. On the client, you can send messages to the `ServiceWorker` like below.

```js
<mark>swivel.emit</mark>('remove-cache', 'v1');
```

Then on the worker, you could just listen for that with a matching API.

```js
</mark>swivel.on</mark>('remove-cache', (context, key) => caches.delete(key));
```

If the worker needs to reply, it can use `context.reply`.

```js
swivel.on('remove-cache', function handler (context, key) {
  caches.delete(key).then(function () {
      <mark>context.reply</mark>('removed-cache', 'ok', 'whatever');
  });
});
```

The client that sent the message then gets to handle the reply as long as they're listening for the `removed-cache` event. Note how the API here is identical to the API for listening on events on the ServiceWorker.

```js
swivel.on('removed-cache', function handler (context, success, metadata) {
  // do something else
});
```

When `ServiceWorker` has important announcements, it can broadcast to every client.

```js
<mark>swivel.broadcast</mark>('announcement', { super: '!important' });
```

Broadcasted messages can be listened using the exact same `swivel.on` API in the client-side. In addition to `swivel.on`, there's also `swivel.once` that binds one time event handlers, and `swivel.off` to remove event handlers.

Lastly, as we mentioned earlier you can interact with different ServiceWorker instances. Instead of using the `swivel.*` API directly, you could use `swivel.at(worker).*` instead. Messages sent from a client using `swivel.at(worker).emit` will only go to `worker`, and messages broadcasted by `worker` will only be available to listeners registered using `swivel.at(worker).on`. This kind of scoping helps prevent accidents when phasing out old workers and when installing new ones.

You can check out the [full documentation on GitHub][1].

> `ServiceWorker`, `MessageChannel`, & `postMessage`. Oh, my!

[1]: https://github.com/bevacqua/swivel "bevacqua/swivel on GitHub"
[2]: https://github.com/bevacqua/swivel#swivelatworker "swivel.at API documentation"
[3]: http://alistapart.com/article/application-cache-is-a-douchebag "Application Cache is a Douchebag"
[4]: /articles/serviceworker-revolution "ServiceWorker: Revolution of the Web Platform on Pony Foo"
[5]: https://i.imgur.com/Svqju4J.gif
