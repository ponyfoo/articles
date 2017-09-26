# Script Tags and Closures

In the early days, JavaScript was inlined in HTML `<script>` tags. At best, it was offloaded to dedicated script files, all of which shared a global scope.

Any variables declared in one of these files or inline scripts would be imprinted on the global `window` object, creating leaks across entirely unrelated scripts that might've lead to conflicts or even broken experiences, where a variable in one script might inadvertently replace a global that another script was relying on.

Eventually, as web applications started growing in size and complexity, the concept of scoping and the dangers of a global scope became evident and more well-known. Immediately-invoking function expressions (IIFE) were invented and became an instant mainstay. An IIFE worked by wrapping an entire file or portions of a file in a function that executed immediately after evaluation. Each function in JavaScript creates a new level of scoping, meaning `var` variable bindings would be contained by the IIFE. Even though variable declarations are hoisted to the top of their containing scope, they'd never become implicit globals, thanks to the IIFE wrapper, thus suppressing the brittleness of implicit JavaScript globals.


Several flavors of IIFE can be found in the next example snippet. The code in each IIFE is isolated and can only escape onto the global context via explicit statements such as `window.fromIIFE = true`.

```js
(function() {
  console.log('IIFE using parenthesis')
})()

~function() {
  console.log('IIFE using a bitwise operator')
}()

void function() {
  console.log('IIFE using the void operator')
}()
```

Using the IIFE pattern, libraries would typically create modules by exposing and then reusing a single binding on the `window` object, thus avoiding global namespace pollution. The next snippet shows how we might create a `mathlib` component with a `sum` method in one of these IIFE-based libraries. If we wanted to add more modules to `mathlib`, we could place each of them in a separate IIFE which adds its own methods to the `mathlib` public interface, while anything else could stay private to the component that defined the new portion of functionality.

```js
void function() {
  window.mathlib = window.mathlib || {}
  window.mathlib.sum = sum

  function sum(...values) {
    return values.reduce((a, b) => a + b, 0)
  }
}()

mathlib.sum(1, 2, 3)
// <- 6
```

This pattern was, coincidentally, an open invitation for JavaScript tooling to burgeon, allowing developers -- for the first time -- to safely concatenate every IIFE module into a single file, reducing the strain on the network.

The problem in the IIFE approach was that there wasn't an explicitly dependency tree. This means developers had to manufacture component file lists in a precise order, so that dependencies would load before any modules that dependend on them did -- recursively.
