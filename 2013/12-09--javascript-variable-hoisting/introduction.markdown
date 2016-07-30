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
