# Progressive Networking

I firmly believe the ["Cached then Network and `postMessage`"][1] strategy I published earlier is one of the best approaches to leveraging `ServiceWorker`.

The killer feature in `ServiceWorker` is offline "first" -- even if I'd strongly prefer calling it something else, like **progressive networking** or whatever. It's not really offline _"first"_, because ServiceWorker has to be installed, thus JavaScript has to run. That means server-side rendering on initial page load is still just as important, and it also means there's nothing _"first"_ about offline first. `</rant>`

As [Jake Archibald][1] often insists on, going to the network first may result in _a very poor experience_ for clients with very low connectivity that are nevertheless considered to be "online" by web browsers -- even though they're barely able to download any content. Timeouts are a very poor solution to the _"barely online"_ issue, and serving whatever content we have in the cache immediately is a way better alternative. After all, the "instant" aspect is what makes **progressive networking** a killer feature. The simple explanation is that that's what happens in native apps, and what humans have eventually come to expect in mobile browsers _(and will soon come to expect from desktop applications as well, because transitivity)_.

# Updating Stale Cached Content

If we want to be strict about progressive networking, that means we're always going to hit the cache, and if there's a hit we're going to serve that. No matter what. Anything, immediately. Way better than the right stuff, but later. The problem, then, is that the cached content may be stale. The strategy I proposed last week was as follows.

- Query the cache, if we hit something, return that immediately
- Query the network, even if the cache was hit
  - Use the network response if the cache was a miss
  - <mark>Notify clients about the updated content if the cache was a hit</mark>

Today I want to focus on that last bullet point.

First, we probably need to decide what warrants an update. Unfortunately, that's mostly application-specific. For example, an image editing website may consider images to be critical content and would thus like to update images whenever they're revalidated in the `ServiceWorker` cache. In the case of this measly blog, as a case study, the content that changes most frequently are comments, and occasionally articles are edited, too.

> The single most important thing we should update in most applications is **views**. Unfortunately, the mechanism for doing that is application-specific _-- framework-specific at best._

Suppose a web page fetches a model via JSON through some kind of client-side router, and then renders a view. In our current situation, that'd be almost instantaneous if an active `ServiceWorker` had a cached copy of the response to that request. Later, the `ServiceWorker` would get the response from the network and update the cache. By that time, though, our client happily received and rendered the _(now stale, maybe)_ previously cached response.

Here's something like what we used to have. The cached response is returned immediately if available, otherwise we return the networked promise. Once the `fetch` response comes through, we store it in the cache.

```js
function queriedCache (cached) {
  var networked = fetch(request)
    .then(fetchedFromNetwork, unableToResolve)
    .catch(unableToResolve);
  return cached || networked;

  function fetchedFromNetwork (response) {
    var cacheCopy = response.clone();
    caches.open(version + 'pages').then(function add (cache) {
      cache.put(request, cacheCopy);
    });
    return response;
  }
}
```

We need to reconfigure the `ServiceWorker`, letting clients know when a previously cached response has been updated. In the `fetchedFromNetwork` callback we could test to see if `cached` is truthy, whether the request matches our origin, and whether the response is JSON. This is specific to the case study, you might need to filter on something else, as to avoid treating `GET` API calls as "view updates". You might also want to check whether the responses are actually different in some way _-- maybe just content length, for instance_ -- so you don't waste your clients time.

```js
var url = new URL(request.url);
var json = response.headers.get('Content-Type').indexOf('application/json') !== -1;
if (cached && url.origin === location.origin && json) {
  // let the client know we have an updated response
}
```

A generalized approach such as this is much more effective than trying to modularize these efforts into updates that are specific to particular views or components, as that'd involve significant more effort in both implementation and future maintainability. More focused approaches are probably better in terms of UX, and as with all things, user testing is king -- or, should be. After all, you're reading this because you care about your humans and that they get the content they deserve, fast and fresh.

Note also that you should `.clone` another copy of the response for messaging purposes. The original goes to the client, the first copy goes to the cache, and the second one is for reading and passing along to clients looking for updates. How do you pass that along exactly? We'll cover that in the next section, for now we'll just refer to the [`swivel`][3] library I made that simplifies and unifies the messaging API.

The following bit of code assumes you've created a copy of the response like `updateCopy = response.clone()`. For performance reasons, it's better to try and extract the logic around figuring out whether you'll actually need to notify clients, as that way you can clone the response only in those cases, reducing strain on the CPU.

```js
var url = new URL(request.url);
var json = response.headers.get('Content-Type').indexOf('application/json') !== -1;
if (cached && url.origin === location.origin && json) {
  <mark>updateCopy.json()</mark>.then(function parsed (data) {
    <mark>swivel.broadcast</mark>('view-update', request.url, data);
  });
}
```

In an ideal world, we'd unicast the updated view model to the client who made the request, instead of broadcasting every client, but alas, the mechanism to associate `fetch` requests with specific clients is still a moving part of the `ServiceWorker` specification and hasn't been implemented in browsers yet.

On the client-side, the web pages, we can now register a [`swivel`][3] event handler for `view-update` that refreshes the view. Again, we'll discuss later how `swivel` works under the hood.

```js
navigator.serviceWorker
  .register('/service-worker.js')
  .then(navigator.serviceWorker.ready)
  .then(setupMessaging);

function setupMessaging () {
  swivel.on('view-update', function renderUpdate (context, href, data) {
    // use data to re-render view
  });
}
```

When it comes to updating the view, that's up to the implementation. If you're using a framework like React or Taunus, it's a really easy thing to do, you just re-apply the model to the view component and let the framework render that. Updating server-rendered HTML is a bit trickier, as the updated response is also HTML, and the client-side JavaScript probably hasn't been executed yet when the `fetch` response comes through, meaning you'll have to use a different mechanism to update views in this case.

A possible solution might be to take the opposite route: have the client ask the `ServiceWorker` for an update from the cache as soon as the client-side JavaScript executes. If your view rendering framework is flexible enough, you could even reuse the `renderUpdate` method by having the `ServiceWorker` reply with an updated JSON view.

On the client-side, when the page first loads, you could run the following code, telling the ServiceWorker you want an update on the cached content that was just rendered.

```js
swivel.emit('active-client', location.href);
```

The `ServiceWorker` could then listen for these messages and reply with a JSON response for that page.

```js
swivel.on('active-client', function activeClient (context, href) {
  caches
    .match(href)
    .then(function (response) {
      return response.json();
    }).then(function (data) {
      <mark>context.reply('view-update', href, data)</mark>;
    });
});
```

Then the client, in turn, applies the changes to the view as needed. Note that you don't necessarily have to immediately update the view. Even if you're using React or some other virtual DOM powered engine, the UX may be weird if you just update the site, and you may want to consider partially applying updates or even showing a message indicating there's more content, and the human could click that message to get the updates -- for example. As usual, there's a ton of options here.

What do you think would work best in your use case? I'd love to hear some opinions on this subject.

[1]: /articles/serviceworker-revolution#cached-then-network-and-postmessage "Cached then Network and postMessage - ServiceWorker: Revolution of the Web Platform on Pony Foo"
[2]: https://twitter.com/jaffathecake "@jaffathecake on Twitter"
[3]: https://github.com/bevacqua/swivel "bevacqua/swivel on GitHub"
