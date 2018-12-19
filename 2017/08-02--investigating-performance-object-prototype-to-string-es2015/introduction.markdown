# Introduction

The [ECMAScript 2015 Language Standard](http://ecma-international.org/ecma-262/6.0/) introduced the concept of so-called [well-known symbols](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Symbol#Well-known_symbols) to the JavaScript language. These are special built-in symbols which represent internal language behaviors that were not exposed to developers in ECMAScript 5 and earlier. Examples of these are:

* [`Symbol.iterator`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/iterator): A method returning the default iterator for an object. Used by [`for..of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of), [`yield*`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*), the [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator), [destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), etc.

* [`Symbol.hasInstance`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/hasInstance): A method for determining if a constructor object recognizes an object as its instance. Used by the [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) operator.

* [`Symbol.toStringTag`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag): A string value used for the default description of an object, which is consulted by the [`Object.prototype.toString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) method.

Most of these newly introduced symbols affect several parts of the JavaScript language in non-trivial and cross-cutting ways, and lead to significant changes in the performance profile due to the additional [monkey-patchability](https://en.wikipedia.org/wiki/Monkey_patch). Operations that were not observable by JavaScript code are all of a sudden observable and the behavior of these operations can be changed by user code.

One particularly interesting example of this is the new [`Symbol.toStringTag`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toStringTag) symbol, which is used to control the behavior of the [`Object.prototype.toString()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString) built-in method. For example, a developer can now put this special property on any instance, and it is then used instead of the default built-in tag when the `toString` method is invoked:

```javascript
class A {
  get [Symbol.toStringTag]() { return 'A'; }
}
Object.prototype.toString.call(‘’);     // "[object String]"
Object.prototype.toString.call({});     // "[object Object]"
Object.prototype.toString.call(new A);  // "[object A]"
```

This requires that the implementation of `Object.prototype.toString()` for ES2015 and later now converts its **_this_** *value* into an object first via the abstract operation [ToObject](https://tc39.github.io/ecma262/#sec-toobject) and then looks for `Symbol.toStringTag` on the resulting object and in its prototype chain. The [relevant part](https://tc39.github.io/ecma262/#sec-object.prototype.tostring) of the language specification looks like this:

![Object.prototype.toString ()][1]

Here you can see the [`ToObject`](https://tc39.github.io/ecma262/#sec-toobject) conversion as well as the [`Get`](https://tc39.github.io/ecma262/#sec-get-o-p) for `@@toStringTag` (this is special internal syntax for the language specification for the well-known symbol with the name *toStringTag*). The addition of `Symbol.toStringTag` in ES2015 adds a lot of flexibility for developers, but at the same time comes at a cost.

  [1]: https://images.ponyfoo.com/uploads/object-prototype-tostring-8d733121570a47d4ae2132c85a59f36f.png "object-prototype-tostring.png"
