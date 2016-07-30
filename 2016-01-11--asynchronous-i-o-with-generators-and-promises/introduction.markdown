Coming up with practical code examples to keep a book interesting is -- for me -- one of the hardest aspects of writing engaging material. I find that the best examples are the ones that get you thinking about API design and coding practices, beyond just explaining what a specific language feature does. That way, if you already understand the language feature at hand, you might still find the practical thought exercise interesting.

The example in question involved finding a use case for `return` in a generator function. [As we know][1], generators treat `return` statements differently from `yield` expressions. Take for example the following generator.

```js
function* numbers () {
  yield 1;
  yield 2;
  return 3;
  yield 4;
}
```

If we use `Array.from(numbers())`, `[...numbers()]`, or even a `for..of` loop, we'll only ever see `1` and `2`. However, if we went ahead and used the generator object, we'd see the `3` as well -- although the iterator result would indicate `done: true`.

```js
var g = numbers();
console.log(g.next());
// <- { done: false, value: 1 }
console.log(g.next());
// <- { done: false, value: 2 }
console.log(g.next());
// <- { <mark>done: true</mark>, value: 3 }
```

The example I came up with involved a function call passing in a generator, where you `yield` resources that should be persisted, and then you `return` the endpoint where you'd like to persist those resources. The iterator would then pull each resource at a time, and finally push the data for each resource to another endpoint, which would presumably save all that data in an object.

[1]: /articles/es6-generators-in-depth "ES6 Generators in Depth on Pony Foo"
[2]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
