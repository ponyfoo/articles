Let's do a quick recap of generators _[(read our primer on generators here)][generators]_. Generator functions return generator objects when invoked. A generator object has a `next` method, which returns the next element in the sequence. The `next` method returns objects with a `{ value, done }` shape.

The following example shows an infinite fibonacci number generator. We then instantiate a generator object and read the first eight values in the sequence.

```js
function* fibonacci() {
  let previous = 0
  let current = 1
  while (true) {
    yield current
    const next = current + previous
    previous = current
    current = next
  }
}
const g = fibonacci()
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 2, done: false }
console.log(g.next()) // <- { value: 3, done: false }
console.log(g.next()) // <- { value: 5, done: false }
console.log(g.next()) // <- { value: 8, done: false }
console.log(g.next()) // <- { value: 13, done: false }
console.log(g.next()) // <- { value: 21, done: false }
```

[generators]: /articles/es6-generators-in-depth "ES6 Generators in Depth on Pony Foo"
