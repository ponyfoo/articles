
Event emitters usually support multiple types of events, rather than a single one. Let's implement, step by step, our own function to create event emitters, or improve existing objects as event emitters. In a first step, I'll either return the object unchanged, or create a new object if one wasn't provided.

```js
function emitter (thing) {
  if (!thing) {
    thing = {};
  }
  return thing;
}
```

Being able to use multiple event types is powerful and only costs us an object to store the mapping of event types to event listeners. Similarly, we'll use an array for each event type, so that we can bind multiple event listeners to each event type. I'll also add a simple function which registers event listeners while I'm at it.

```js
function emitter (thing) {
  var events = {};

  if (!thing) {
    thing = {};
  }

  thing.on = function (type, listener) {
    if (!events[type]) {
      events[type] = [listener];
    } else {
      events[type].push(listener);
    }
  };

  return thing;
}
```

So far so good, now you can add event listeners, once an emitter is created. This is how it'd work. Keep in mind that listeners can be provided with an arbitrary number of arguments, when an event is fired, and we'll implement the method to fire events next.

```js
var thing = emitter();

thing.on('change', function () {
  console.log('thing changed!');
});
```

Naturally, that works just like a DOM event listener. All we need to do now is implement the method which fires the events. Without it, there wouldn't be an event emitter. I'll implement an `emit` method which allows you to fire the event listeners for a particular event type, passing in an arbitrary number of arguments. Here is how it'd look like.

```js
thing.emit = function (type) {
  var evt = events[type];
  if (!evt) {
    return;
  }
  var args = Array.prototype.slice.call(arguments, 1);
  for (var i = 0; i < evt.length; i++) {
    evt[i].apply(thing, args);
  }
};
```

The `Array.prototype.slice.call(arguments, 1)` statement is an interesting one. Here I'm apply `Array.prototype.slice` on the `arguments` object, and telling it to start at index 1. This does two things for me. It casts the arguments object into a true array, and it gives me a nice array with all of the arguments that were passed into `emit`, except for the event type, which I don't need to invoke the event listeners.

There's one last tweak I'd like to do, which is executing the listeners asynchronously, so that they don't halt execution of the main loop if one of them blows up. You could also use a try catch block here, but I'd rather not get involved with exceptions in event listeners, let the consumer handle that. To achieve this, I'll just use a `setTimeout` call, as shown below.

```js
thing.emit = function (type) {
  var evt = events[type];
  if (!evt) {
    return;
  }
  var args = Array.prototype.slice.call(arguments, 1);
  for (var i = 0; i < evt.length; i++) {
    debounce(evt[i]);
  }
  function debounce (e) {
    setTimeout(function () {
      e.apply(thing, args);
    }, 0);
  }
};
```

You should now be able to create emitter objects, or you can also turn existing objects into event emitters. Note that, because I'm debouncing the event listeners, if an event throws the rest _will still run to completion_. This is not always the case in other implementations of events.

#### Emitters inside `contra`

If you check out the documentation for [contra][1] you'll find out that the interface to interact with `λ.emitter` is basically the same as what I've just explained. In addition to the `on()` and `emit()` methods, the implementation in `contra` offers a `once()` method which would register an event handler that should only trigger once, and an `off()` method which can turn off any listener, including those registered by `once()`, all while staying [around 30 lines][2] of code!

[1]: https://github.com/bevacqua/contra "Contra: Asynchronous flow control with a functional taste to it"
[2]: https://github.com/bevacqua/contra/blob/master/src/contra.js#L140-L171 "Contra's implementation of Event Emitters"
