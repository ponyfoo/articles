# `Observable` and the `observer` API

In this proposal, `Observable` would be a new built-in that can be used to handle event streams. The `Observable` constructor takes a callback which defines an event stream. In the following example, our observable returns a stream of events with just `1` and `2` values. The `observer.next` method can be used to add events to the observable stream.

```js
new Observable(observer => {
  observer.next(1)
  observer.next(2)
})
```

We can use `observer.error` to add report errors that occur during stream processing.

```js
new Observable(observer => {
  observer.error(new Error(`Failed to stream events`))
})
```

We can use `observer.complete` to signal when the event stream has come to an end.

```js
new Observable(observer => {
  observer.next(1)
  observer.next(2)
  observer.complete()
})
```

The callback passed to an `Observable` constructor can return a cleanup function to tear down our observable. This can be useful to remove event listeners, clear timeouts, and similar cleanup tasks. The following observable is a bit more interesting than the last one. It tracks mouse position relative to the page as the user moves the cursor on screen, producing an event stream that describes the cursor position on the page through time.

```js
function mouseTracking () {
  return new Observable(observer => {
    const handler = ({ pageX, pageY }) => {
      observer.next({ x: pageX, y: pageY })
    }

    document.body.addEventListener(`mousemove`, handler)

    return () =>  {
      document.body.removeEventListener(`mousemove`, handler)
    }
  })
}
```

In order for us to subscribe to an observable event stream, we just call the `Observable#subscribe` method on an observable object instance. Doing so will invoke the callback passed to the `Observable` constructor in the previous code snippet, binding the event listener and getting the event stream started. Moving the mouse will now result in events being fired into the event stream.

```js
mouseTracking().subscribe({
  next({ x, y }) { console.log(`New position: ${ x }, ${ y }`) },
  error(err) { console.log(`Error: ${ err }`) },
  complete() { console.log(`Done!`) }
})
```

# `Subscription#unsubscribe`

`Observable#subscribe` returns an object that lets us `unsubscribe`, executing the cleanup method *-- if one exists*. When we're no longer interested in events from the observable stream, we'll unsubscribe and let the observable clean itself up.

```js
const subscription = mouseTracking().subscribe({
  next({ x, y }) { console.log(`New position: ${ x }, ${ y }`) },
  error(err) { console.log(`Error: ${ err }`) },
  complete() { console.log(`Done!`) }
})

subscription.unsubscribe()
```

# `Observable.of`

`Observable.of(...items)` is a simple static utility helper that creates an `Observable` out of the provided `items`. The `items` are then delivered synchronously when `Observable#subscribe` is called.

```js
Observable.of(1, 2, 3, 4).subscribe({
  next(item) { console.log(item) }
})
// <- 1
// <- 2
// <- 3
// <- 4
```

We can think of `Observable.of` as the following *simplified* implementation, where we return a synchronous stream of provided values.

```js
Observable.of = (...items) => {
  return new Observable(observer => {
    items.forEach(item => {
      observer.next(item)
    })
    observer.complete()
  })
}
```

# `Observable.from`

This static method casts the provided argument into an `Observable`. If the provided `Object` has a `Symbol.observable` method, then the result of invoking that method is returned.

```js
Observable
  .from({
    [Symbol.observable]() { return Observable.of(1, 2, 3) }
  })
  .subscribe({
    next(item) { console.log(item) }
  })
// <- 1
// <- 2
// <- 3
```

If the provided argument doesn't implement a `Symbol.observable` method, then it's assumed to be an iterable. A synchronous `Observable` sequence of the iterable is returned.

```js
Observable
  .from([1, 2, 3])
  .subscribe({
    next(item) { console.log(item) }
  })
// <- 1
// <- 2
// <- 3
```

In this case, `Observable.from` is similar to `Observable.of`. We could think of `Observable.from` as the following *simplified* implementation.

```js
Observable.from = value => {
  if (typeof value[Symbol.observable] === `function`) {
    return value[Symbol.observable]()
  }
  return Observable.of(Array.from(value))
}
```

# Conclusions

Note that this proposal is still in its infancy, but it'd lay out the foundation for functional programming at the native JavaScript level. Eventually, it may earn the ability to `Observable#filter` or `Observable#map` the stream of events, allowing us to focus only on the kinds of events we want to listen for.

In the meantime, these could be implemented in user-land as we continue to watch out for patterns and let the specification evolve naturally and gradually. You can find [a polyfill for the current incarnation][polyfill] of the specification on the GitHub repository, but you'll have to delete the `export` keyword if you want to play around with it in your browser's Dev Tools.

[polyfill]: https://github.com/tc39/proposal-observable/blob/0fa13995f372bab50de8cb5e8db59066ad08dd7a/src/Observable.js "Observable.js polyfill on GitHub"
