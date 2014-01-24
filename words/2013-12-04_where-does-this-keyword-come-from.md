# Where does this keyword come from?

Working on the latest chapter for my upcoming book on [JavaScript Application Design](http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"), I'm writing about how scoping works. I want to share a particular code sample which I hope will bring some clarity to how `this` works. It's not all dark magic, learning about `this` can be _tremendously helpful_ to your development as a JavaScript programmer.

Until you _"get it"_, this is probably how you feel about `this`.

![chaos.gif][1]

It's madness, right? In this brief article, I aim to demystify `this`.

  [1]: https://raw.github.com/bevacqua/buildfirst/master/images/chaos.gif
  
## How `this` works

If the method is invoked on an object, that object will be assigned to `this`.

```js
var parent = {
    method: function () {
        console.log(this);
    }
};

parent.method();
// <- parent
```

Note that this behavior is very _"fragile"_, if you get a reference to method, and invoke that, then `this` won't be `parent` anymore, but rather the `window` global object once again. This confuses most developers.

```js
var parentless = parent.method;

parentless();
// <- Window
```

The bottom line is you should look at the call site to figure out whether the function is invoked as a property of an object or on its own. If its invoked as a property, then that property will become `this`, otherwise `this` will be assigned the value of the global object, or `window`. In this case, but under [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode "Strict mode explained on MDN"), `this` will be `null` instead.

In the case of constructor functions, `this` is assigned to the instance that's being created, when using the `new` keyword.

```js
function ThisClownCar () {
  console.log(this);
}

new ThisClownCar();
// <- ThisClownCar {}
```

Note that this behavior doesn't have a way of telling a function is supposed to be used as a constructor function, and thus omitting the new keyword will result in `this` being the global object, like we saw in the `parentless` example.

```js
ThisClownCar();
// <- Window
```

## Tampering with `this`

The `.call`, `.apply`, and `.bind` methods are used to manipulate function invocation, helping us to define both the value for `this`, and the `arguments` provided to the function.

`Function.prototype.call` takes any number of arguments, the first one is assigned to `this`, and the rest are passed as arguments to the function that's being invoked.

```js
Array.prototype.slice.call([1, 2, 3], 1, 2)
// <- [2]
```

`Function.prototype.apply` behaves very similarly to `.call`, but it takes the arguments as a single array with every value, instead of any number of parameter values.

```js
String.prototype.split.apply('13.12.02', ['.'])
// <- ['13', '12', '02']
```

`Function.prototype.bind` creates a special function which can be used to invoke the function it is called on. That function will always use the `this` argument passed to `.bind`, as well as being able to assign a few arguments, creating a [curried version](http://en.wikipedia.org/wiki/Currying "Currying on Wikipedia") of the original function.

```js
var arr = [1, 2];
var add = Array.prototype.push.bind(arr, 3);

// effectively the same as arr.push(3)
add();

// effectively the same as arr.push(3, 4)
add(4);

console.log(arr);
// <- [1, 2, 3, 3, 4]
```

## Scoping `this`

In the next case, `this` will stay the same across the scope chain, this is the exception to the rule, and often leads to confusion among amateur developers.

```js
function scoping () {
  console.log(this);

  return function () {
    console.log(this);
  };
}

scoping()();
// <- Window
// <- Window
```

A common work-around is to create a local variable which holds onto the reference to `this`, and isn't shadowed in the child scope. The child scope shadows `this`, making it impossible to access a reference to the parent `this` directly.

```js
function retaining () {
  var self = this;

  return function () {
    console.log(self);
  };
}

retaining()();
// <- Window
```

Unless you really want to use both the parent scope's `this`, as well as the current value of `this` for some obscure reason, the method I prefer is to use the `.bind` function. This can be used to assign the parent `this` to the child scope.

```js
function bound () {
  return function () {
    console.log(this);
  }.bind(this);
}

bound()();
// <- Window
```

Have you ever had any problems with this? How about `this`? Let me know if you think I've missed any other edge cases or elegant solutions.
