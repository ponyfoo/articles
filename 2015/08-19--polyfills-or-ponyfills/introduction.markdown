These days we all know what polyfills are. It's been over five years since Remy Sharp [coined the term][1]. A polyfill is usually a snippet of code that patches a piece of functionality that's missing in some browsers. With ES5, polyfills became all the rage because you could instantly get access to functional `Array.prototype` methods like `.map` and `.reduce` just by dropping in a file. There's also entire bundles that patch most of ES5 for you to use in older browsers, such as [`es5-shim`][2]. However, not all is peaches and cream.

When it comes to ES6, a flurry of problems turn polyfills into ineffective vaccines. For one, you simply **can't polyfill language features**, such as arrow functions, generators, `async`/`await` _(ES7)_, rest and spread parameters, classes, modules, etc. There are other features you _\*could\*_ actually polyfill, such as [`Array.of`][3], [`Number.isNaN`][4] or [`Object.assign`][5], because those don't introduce syntax changes to the language -- except that **you shouldn't**.

[1]: https://plus.google.com/+PaulIrish/posts/4okUyAE1qQH
[2]: https://github.com/es-shims/es5-shim
[3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/of
[4]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/isNaN
[5]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
