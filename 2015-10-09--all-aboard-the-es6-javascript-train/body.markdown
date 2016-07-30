# What about ES6 and Node.js?

On the server-side, you _(as opposed to your customers when it comes to browsers)_ are in charge of picking the runtime version. The most recent versions of Node.js have aggressively updated their dependency on `v8`, the JavaScript engine that powers Google Chrome, to the latest and greatest. That means if you're using one of the latest versions of Node.js _(`v4.x.x` at the time of this writing)_, you have all the same ES6 goodies that are available in your Google Chrome browser.

Sometimes you want to keep your existing version of Node.js for most projects but also want to use a recent version because of ES6. In those cases, you can use [`nvm`](https://github.com/creationix/nvm "creationix/nvm on GitHub") to manage multiple versions of Node.js, allowing you to keep your existing projects intact while adopting the latest ES6 features in your newer projects. It's easy to use `nvm`, you just open up a command-line, type commands such as `nvm install $VERSION` or `nvm use $VERSION`, and you're set.

Juggling many different versions of `node` may prove to be too much of a hassle to you. An alternative may be to use `babel-node`, a CLI utility that transpiles your ES6 code before executing it through `node`. You can install `babel-node`, along with the standalone Babel transpiler, using the command shown below.

```js npm install babel --global ```

While `babel-node` works great in development, it's not recommended for production as it can introduce significant lag in startup time. A better approach might be to precompile your modules before every deployment. That way your application won't need to recompile itself at startup.

![A babel-node CLI demonstration][1]

<sub>Using babel-node can add ES6 support to older versions of Node.js. Upgrading to the latest version of Node.js, where possible, is always recommendable.</sub>

The latest version of the language includes features such as arrow functions, template literals, block scoping, collections, native promises, and many more. I can't possibly cover all of it in one go, but we'll go over my favorites: the most practical.

## Template Literals

In ES6 we get templates that are similar to those in Mustache, but native to JavaScript. In their purest form, they are almost indistinguishable from regular JavaScript strings, except that they use backticks as a delimiter instead of single or double quotes. All the strings found below are equivalent.

```js
"The quick brown fox jumps over the lazy dog"
'The quick brown fox jumps over the lazy dog'
`The quick brown fox jumps over the lazy dog`
```

Backticks aren't convenient just because they rarely appear in text. These strings can be multiline without any extra work on your behalf! That means you no longer need to concatenate strings using `+` or join them using an array.

```js
var text = `This is
  a multi-line
      string.`;
```

Template literals also let you add variables into the mix by wrapping them in an interpolation, as seen below.

```js
var color = 'brown';
var target = 'lazy dog';
`The quick ${color} fox jumps over the ${target}`;
// <- 'The quick brown fox jumps over the lazy dog'
```

You're not limited to variables, you can interpolate any expression in your templates.

```js
var input = -10;
var modifier = 2;
`The result of multiplying ${input} by ${modifier} is ${input * modifier}`;
// <- 'The result of multiplying -10 by 2 is -20'
`The absolute value for ${input} is ${Math.abs(input)}`;
// <- 'The absolute value for -10 is 10'
```

Let's move onto a some other practical features.

## Block Scoping, `let`, and `const`

The `let` statement is one of the most well-known features in ES6. It works like a `var` statement, but it has different scoping rules. JavaScript has always had a complicated ruleset when it came to scoping, driving many programmers crazy when they were first trying to figure out how variables work in the language. Declarations using `var` are function-scoped. That means `var` declarations are accessible from anywhere in the function they were declared in. On the other hand, `let` declarations are _block.scoped_. Block scoping is new to JavaScript in ES6, but it's fairly commonplace in other languages, like Java or C#.

Let's look at some of the differences. If you had to declare a variable using `var` in a code branch for an `if`, your code would look like in the following snippet.

```js
function sortCoordinates (x, y) {
  if (y > x) {
    var temp = y;
    y = x;
    x = temp;
  }
  return [x, y];
}
```

That represents a problem because -- as we know -- the process known as hoisting means that the declaration for `temp` will be "pulled" to the top of its scope. Effectively, our code behaves as if we wrote the snippet below. For this reason, `var` is ineffective when dealing with variables that were meant to be scoped to code branches.

```js
function sortCoordinates (x, y) {
  var temp;
  if (y > x) {
    temp = y;
    y = x;
    x = temp;
  }
  return [x, y];
}
```

The solution to that problem is using `let`. A `let` declaration is also hoisted to the top of its scope, but its scope is the immediate block (denoted by the nearest pair of brackets), meaning that hoisting won't result in behavior you may not expect or variables getting mixed up.

Even though `let` declarations are still hoisted to the top of their scope, attempts to access them in any way before the actual `let` statement is reached will throw. This mechanism is known as the "temporal dead zone". Most often than not, this will catch errors in user code rather than represent a problem, though. Attempts to access variables before they were declared usually lead to unexpected behavior when using `var`, so it's a good thing that `let` prevents it entirely.

```js
console.log(there); // <- runtime error, temporal dead zone
let there = 'dragons';
```

In addition to `let` declarations, we can also observe `const` declarations being added to the language. In contrast with `var` and `let`, `const` declarations must be assigned to upon declaration.

```js
const pi = 3.141592653589793;
const max; // <- SyntaxError
```

Attempts to assign to a different value to a `const` variable will result in syntax errors, as the compiler is able to tell that you're trying to assign to a `const` variable.

```js
const max = 123;
max = 456; // <- SyntaxError
```

Note that `const` only means that the declaration is a _constant reference_, but it doesn't mean that the referenced object becomes immutable. If we assign an object to a `const` variable `client`, we won't be able to change `client` into a reference to something else, but we _will_ be able to change properties on `client` like we're used to with other declaration styles.

```js
const client = getHttpClient('http://ponyfoo.com');
client.maxConcurrency = 3;
// works because we're not assigning to client
```

## Arrow Functions

These are probably the best known feature in ES6. Instead of declaring a function using the `function` keyword, you can use the _"arrow"_ notation, as seen below. Note how the `return` is implicit, as well as the parenthesis around the `x` parameter.

```js
[1, 2].map(function (x) { return x * 2 }); // ES5
[1, 2].map(x => x * 2); // ES6
```

Arrow functions have a flexible syntax. If you have a single parameter you can get away without the parenthesis, but you'll need them for methods with zero, two, or more parameters. Note that you can still use the parenthesis when you have a single parameter, but it's more concise to omit them.

```js
[1, 2].map((x, i) => x * 2 + i); // <- [2, 5]
```

If your method does more than return the results of evaluating an expression, you could wrap the right-hand part of the declaration in a block, and spend as many lines as you need. If that's the case, you'll need to add the `return` keyword back again.

```js
x => {
  // multiple statements
  return result
}
```

An important aspect of arrow functions is that they are lexically scoped. That means you can kiss your `var self = this` statements goodbye. The example below increases a counter and prints the current value, every second. Without lexical scoping you'd have to `.bind` the method call, use `.call`, `.apply`, or the `self` hack we mentioned earlier.

```js
function count () {
  this.counter = 0;
  setInterval(() => console.log(++this.counter), 1000);
}
count(); // <- 1, .. 2, .. 3, ...
```

Arrow functions are recommended for short methods, like those typically provided to `.map` and `.filter` iterators.

# Further Reading

Here are some resources that will help you start taking advantage of ES6 today.

## ES6 in Depth

I wrote [a series of over 20 articles](/articles/tagged/es6-in-depth "ES6 in Depth on Pony Foo") that will give you a comprehensive understanding of ES6. Each article covers a specific feature or aspect of the language that changes in ES6, and they're easy to navigate with tons of examples and practical considerations. These articles cover everything we've discussed so far in this article, in addition to promises, rest and spread, iterators, generators, proxies, collections, changes to `Math`, `Number`, `Object`, and `String`, classes, symbols and reflection. It's a great way to dive head first into ES6.

[![Screenshot of article headlines on Pony Foo][2]](/articles/tagged/es6-in-depth "ES6 in Depth on Pony Foo")

<sub>The articles on Pony Foo in the ES6 series cover topics, ranging from getting started to mastering Proxies and Generators.</sub>

If in-depth series are your thing, you might also want to check out these two resources.

- Axel Rauschmayer's [Blog Posts](http://www.2ality.com/search/label/esnext "Articles tagged esnext on 2ality.com") -- While he doesn't have a formal series on ES6, he wrote dozens of detailed articles describing the technical depths of ES6.
- [ES6 in Depth](https://hacks.mozilla.org/category/es6-in-depth/ "ES6 in Depth on Mozilla Hacks") by Mozilla Hacks -- Written by several authors affiliated with Mozilla, it covers many of the same topics as the series on Pony Foo.

## ECMAScript 6 Compatibility Table

When you get started with ES6, you'll quickly realize no browser quite supports ES6 100% in their stable distribution and without any flags. In the meantime, you can leverage [these compatibility tables](http://kangax.github.io/compat-table/es6/ "ES6 compatibility table by @kangax") to understand what features are implemented across what browsers. Alas, these tables are mostly useful for research as your best bet when it comes to using ES6 today is to rely on a transpiler like Babel.

## Quick Start

If you want to get started right away, your best bet might be to hop into the [Babel REPL](http://babeljs.io/repl/) and start toying around with it. They also have [a great article](http://babeljs.io/docs/learn-es2015/ "A detailed overview of ECMAScript 6 features.") for learning the basics about ES6 language features and syntax.

  [1]: https://i.imgur.com/m346jQz.png
  [2]: https://i.imgur.com/miWiB6s.png
