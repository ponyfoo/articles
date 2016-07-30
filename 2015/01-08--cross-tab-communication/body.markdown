Did you know that `localStorage` fires an event? More specifically, it fires an event whenever an item is added, modified, or removed _in another browsing context_. Effectively, this means that whenever you touch `localStorage` in any given tab, all other tabs can learn about it by listening for the `storage` event on the `window` object, like so:

```js
window.addEventListener('storage', function (event) {
  console.log(event.key, event.newValue);
});
```

The `event` object contains a few relevant properties.

Property   | Description
-----------|---------------------
`key`      | The affected key in `localStorage`
`newValue` | The value that is currently assigned to that key
`oldValue` | The value before modification
`url`      | The URL of the page where the change occurred

Whenever a tab modifies something in `localStorage`, an event fires in every other tab. This means we're able to _communicate across browser tabs_ simply by setting values on `localStorage`. Consider the following pseudo_ish_-code example:

```js
var loggedOn;

// TODO: call when logged-in user changes or logs out
logonChanged();

window.addEventListener('storage', updateLogon);
window.addEventListener('focus', checkLogon);

function getUsernameOrNull () {
  // TODO: return whether the user is logged on
}

function logonChanged () {
  var uname = getUsernameOrNull();
  loggedOn = uname;
  localStorage.setItem('logged-on', uname);
}

function updateLogon (event) {
  if (event.key === 'logged-on') {
    loggedOn = event.newValue;
  }
}

function checkLogon () {
  var uname = getUsernameOrNull();
  if (uname !== loggedOn) {
    location.reload();
  }
}
```

The basic idea is that when a user has two open tabs, logs out from one of them, and goes back to the other tab, the page is reloaded and _(hopefully)_ the server-side logic redirects them to somewhere else. The check is being done only when the tab is focused as a nod to the fact that maybe they log out and they log back in immediately, and in those cases we wouldn't want to log them out of every other tab.

We could certainly improve that piece of code, but it serves its purpose pretty well.  A better implementation would probably ask them to log in on the spot, but note that this also works the other way around: when they log in and go to another tab that was also logged out, the snippet detects that change reloading the page, and then the server would redirect them to the logged-in fountain-of-youth blessing of an experience you call your website _(again, hopefully)_.

# A simpler API

The `localStorage` API is arguably one of the easiest to use APIs there are, when it comes to web browsers, and it also enjoys quite thorough cross-browser support. There are, however, some quirks such as incognito Safari throwing on sets with a `QuotaExceededError`, no support for JSON out the box, or older browsers bumming you out.

For those reasons, I put together [local-storage][1] which is a module that provides a simplified API to `localStorage`, gets rid of those quirks, falls back to an in-memory store when the `localStorage` API is missing, and also makes it easier to consume `storage` events, by letting you register and unregister listeners for specific keys.

API endpoints in `local-storage@1.3.1` _(**latest**, at the time of this writing)_ are listed below.

- `ls(key, value?)` gets or sets `key`
- `ls.get(key)` gets the value in `key`
- `ls.set(key, value)` sets `key` to `value`
- `ls.remove(key)` removes `key`
- `ls.on(key, fn(value, old, url))` listens for changes to `key` in other tabs, triggers `fn`
- `ls.off(key, fn)` unregisters listener previously added with `ls.on`

It's also worth mentioning that [local-storage][1] registers a single `storage` event handler and keeps track of every key you want to observe, rather than register multiple `storage` events.

I'd be interested to learn about other use cases for low-tech communication across tabs! Certainly sounds useful for _offline-first_ development, particularly if we keep in mind that `SharedWorker` might take a while to become widely supported, and WebSockets are unreliable in offline-first scenarios.

[1]: https://github.com/bevacqua/local-storage "bevacqua/local-storage on GitHub"
