# JavaScript Variable Hoisting

> A large number of JavaScript interview questions, if not most of them, can be answered with an understanding of scoping, how `this` works, and hoisting.

You might be expecting the method to print `'number'` first, and `2` afterwards, or maybe `3`? Try running it! Why does it print `'undefined'` and then `undefined`? Well, hello hoisting! It'll be easier for you to picture it if I re-arrange the code to how it ends up after hoisting takes place. Let's have a look.

```js
var value = 2;

test();

function test () {
    console.log(typeof value);
    console.log(value);
    var value = 3;
}
```

Enter [JavaScript variable hoisting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Scope_Cheatsheet#Hoisting "Variable Hosting on MDN"), and your code will actually end up looking like below. Hoisting _basically moves variable declarations_ to the top of the scope those variables belong to. However, assignments stay where they are! Function declarations are hoisted only if they're not part of an assignment statement.

```js
var value;

function test () {
    var value;
    console.log(typeof value);
    console.log(value);
    value = 3;
}

value = 2;

test();
```

So, that's why: the `value` declaration at the end of the `test` function actually got hoisted to the top of the scope, and also the reason why `test` didn't meet us with a `TypeError` exception warning us about `undefined` not being a function. Keep in mind that if we used the variable form of declaring the `test` function, we would in fact have gotten that error, because although `var test` would've been hoisted, the assignment wouldn't have been, effectively becoming the following:

```js
var value;
var test;

value = 2;

test();

test = function () {
    var value;
    console.log(typeof value);
    console.log(value);
    value = 3;
};
```

The above wouldn't work as expected, because `test` isn't defined by the time we want to invoke it.

![hoisting.png][1]

### In the real world

Below is a real bug I had to track down once upon a time. Here, the issue was that we were re-declaring the path variable. What's worse, due to hoisting, `path` would become `undefined` if `!something`, resulting in unexpected behavior. It also goes to show how much better it would've been to stick to a pattern where we "hoist" variables ourselves, rather than letting the language to do it for us. Hoisting code would improve the visibility of potential issues such as the one depicted below.

```js
var path = require('path');

// ...

module.exports = function (something) {
  // ...

  if (something) {
    var path = require('path');
    // ...
  }

  // ...
};
```

Granted, it's not a very common issue, but it happens!

  [1]: http://i.imgur.com/eGT7oTe.png "Variable hoisting in action"
