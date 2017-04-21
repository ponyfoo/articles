One way to make this communication pattern effective is using an event emitter (some day [Object.observe](http://updates.html5rocks.com/2012/11/Respond-to-change-with-Object-observe "Respond to change with Object.observe") will be a _realistic_ alternative), and emit events whenever our state changes. In code, we can make our library an event emitter like below. We'll keep state in a separate private variable so that it can't be accessed from outside of our library, making sure we're masters of the universe when it comes to one of our instance's state.

```js
// always use closures, keep it to yourself
(function (window, EventEmitter2) {
  // see: http://stackoverflow.com/q/8651415/389745
  'use strict';

  // we'll keep track of state in a private variable only we can access
  var state = {};

  // we'll use this to give each instance a unique id
  var lastId = -1;

  // this is our Module's constructor
  function Module () {
    var self = this;

    // assign a new unique id
    self._id = ++lastId;
    
    // initialize our state with some defaults
    state[self._id] = {
      x: 0, y: 0
    };
    
    // invoke the EventEmitter2 constructor
    EventEmitter2.call(self, {
      wildcard: true
    });
  }

  // inherit from EE2 prototype, 'subclassing' it
  Module.prototype = Object.create(EventEmitter2.prototype);
  Module.prototype.constructor = Module;

  // expose our Module as some Thing the global object has access to.
  window.Thing = Module;
})(window, EventEmitter2);
```

Then, applying this pattern is simply a matter of exposing commands implementors can interact with (our _public interface_), and emitting changes to our properties, avoiding to _expose these directly_. We will expose commands, such as `move`, where users of our library can change the state of their components in a clearly defined way. They won't be able to access this state, however, except when we choose to expose it, so they need to be paying attention to our _emitted events_.

```js
// the move command
Module.prototype.move = function (x, y) {
  // get the old X and Y states, add some distance to them
  set(this, 'x', get(this, 'x') + x);
  set(this, 'y', get(this, 'y') + y);
};

// internal state setter
function set (instance, key, value) {
  state[instance._id][key] = value;

  // emit events when properties change
  instance.emit(['report', key], value);  
}

// internal state getter
function get (instance, key) {
  return state[instance._id][key];
}
```

Say we were using [keymaster](https://github.com/madrobby/keymaster "keymaster on GitHub") to handle keyboard input, our module could then _consume input_ like below. Note how we don't really even know what we're doing, we are just telling the component:

> Hey, `Thing`!, Obey this command, `.move`!

The code would look something like this.

```js
// get a Thing instance, obviously we'll always want the same one
var component = new Thing();

var map = {
  left: -1,
  right: 1,
  up: -1,
  down: 1
};

// on left or right, move in the X axis
key('left, right', function (e, h) {
  // tell the component to move
  component.move( map[h.shortcut], 0 );
});

// on up or down, move in the Y axis
key('up, down', function (e, h) {
  // tell the component to move
  component.move( 0, map[h.shortcut] );
});
```

Lastly, updating our UI would _also_ be completely decloupled from the rest of the code. Whenever the state changes, we update the position of the UI representation for our `Thing` instance. We don't care _how_ it changed, we only care _that it did_.

```js
// get the element that represents our state in the UI
var element = document.getElementById('square');

// whenever we get a report on X...
component.on('report.x', function (value) {
  // update our left position
  element.style.left = value + 'px';
});

// whenever we get a report on Y...
component.on('report.y', function (value) {
  // update our top position
  element.style.top = value + 'px';
});

// initialize at X=50, Y=50
component.move(50, 50);
```

A [working example](http://cdpn.io/ejBvu "View in CodePen") can be found clicking on the image below.

[![thing.png][2]](http://cdpn.io/ejBvu "View in CodePen")

This kind of pattern might be most useful when your components present _complex interactions between their different properties_. In those cases, it might be even better to use **Angular.js**. Although sometimes a _combination of both_ might also prove to be useful. Of course, the hiding of information is entirely optional, and it might make sense not to hide anything but instead expose the properties in our components directly, however the event emitter pattern still makes sense to "report" _when_ these events take place.

  [2]: https://i.imgur.com/1f66Pk6.png
