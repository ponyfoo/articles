A `const` statement only prevents the variable binding from referencing a different value. Another way of representing that difference is the following piece of code, where we create a `people` variable using `const`, and later assign that variable to a plain `var humans` binding. We can reassign the `humans` variable to reference something else, because it wasn't declared using `const`. However, we can't reassign `people` to reference something else, because it was created using `const`.

```js
const people = ['Tesla', 'Musk']
var humans = people
humans = 'evil'
console.log(humans)
// <- 'evil'
```

If our goal was to make the value immutable, then we'd have to use a function such as `Object.freeze`. Using `Object.freeze` prevents extensions to the provided object, as represented in the following code snippet.

```js
const frozen = Object.freeze(['Ice', 'Icicle', 'Ice cube'])
frozen.push('Water')
// Uncaught TypeError: Can't add property 3, object is not extensible
```
