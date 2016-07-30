# The ES6 Module System

Before ES6 we really went out of our ways to obtain modules in JavaScript. Systems like RequireJS, Angular's dependency injection mechanism, and CommonJS have been catering to our modular needs for a long time now _-- alongside with helpful tools such as Browserify and Webpack_. Still, the year is 2015 and a standard module system was long overdue. As we'll see in a minute, you'll quickly notice that ES6 modules have been heavily influenced by CommonJS. We'll look at [`export`](#export) and [`import`](#import) statements, and see how ES6 modules are compatible with CommonJS, as we'll go over throughout this article.

Today we are going to cover a few areas of the ES6 module system.

- [Strict Mode](#strict-mode)
- [`export`](#export)
  - [Exporting a Default Binding](#exporting-a-default-binding)
  - [Named Exports](#named-exports)
  - [Bindings, Not Values](#bindings-not-values)
  - [Exporting Lists](#exporting-lists)
- [Best Practices and `export`](#best-practices-and-export)
- [`import`](#import)
  - [Importing Default Exports](#importing-default-exports)
  - [Importing Named Exports](#importing-named-exports)
  - [`import` All The Things](#import-all-the-things)

# Strict Mode

In the ES6 module system, strict mode is turned on by default. In case you don't know what strict mode is, it's just a stricter version of the language that disallows lots of bad parts of the language. It enables compilers to perform better by disallowing non-sensical behavior in user code, too. The following is a summary extracted from changes documented in the [strict mode article][1] on MDN.

- Variables can't be left undeclared
- Function parameters must have unique names _(or are considered syntax errors)_
- `with` is forbidden
- Errors are thrown on assignment to _read-only_ properties
- Octal numbers like `00840` are syntax errors
- Attempts to `delete` undeletable properties `throw` an error
- `delete prop` is a syntax error, instead of assuming `delete global[prop]`
- `eval` doesn't introduce _new_ variables into its surrounding scope
- `eval` and `arguments` can't be bound or assigned to
- `arguments` doesn't magically track changes to method parameters
- `arguments.callee` throws a `TypeError`, no longer supported
- `arguments.caller` throws a `TypeError`, no longer supported
- Context passed as `this` in method invocations is not _"boxed" (forced)_ into becoming an `Object`
- No longer able to use `fn.caller` and `fn.arguments` to access the JavaScript stack
- Reserved words _(e.g `protected`, `static`, `interface`, etc)_ cannot be bound

In case it isn't immediately obvious -- you should `'use strict'` in all the places. Even though it's becoming de-facto in ES6, it's still a good practice to use `'use strict'` everywhere in ES6. I've been doing it for a long time and never looked back!

Let's now get into `export`, our first ES6 modules keyword of the day!

# `export`

In CommonJS, you export values by exposing them on `module.exports`. As seen in the snippet below, you could expose anything from a value type to an object, an array, or a function.

```js
<mark>module.exports =</mark> 1
module.exports = NaN
module.exports = 'foo'
module.exports = { foo: 'bar' }
module.exports = ['foo', 'bar']
module.exports = function foo () {}
```

ES6 modules are files that `export` an API -- just like CommonJS modules. Declarations in ES6 modules are scoped to that module, just like with CommonJS. That means that any variables declared inside a module aren't available to other modules unless they're _explicitly exported_ as part of the module's API _(and then imported in the module that wants to access them)_.

## Exporting a Default Binding

You can mimic the CommonJS code we just saw by changing `module.exports =` into `export default`.

```js
<mark>export default</mark> 1
export default NaN
export default 'foo'
export default { foo: 'bar' }
export default ['foo', 'bar']
export default function foo () {}
```

Contrary to CommonJS, `export` statements can only be placed at the top level in ES6 modules -- even if the method they're in would be immediately invoked when loading the module. Presumably, this limitation exists to make it easier for compilers to interpret ES6 modules, but it's also a good limitation to have as there aren't that many good reasons to dynamically define and expose an API based on method calls.

```js
function foo () {
  export default 'bar' // SyntaxError
}
foo()
```

There isn't just `export default`, you can also use _named exports_.

## Named Exports

In CommonJS you don't even have to assign an object to `module.exports` first. You could just tack properties onto it. It's still a single binding being exported _-- whatever properties the `module.exports` object ends up holding._

```js
module.exports.foo = 'bar'
module.exports.baz = 'ponyfoo'
```

We can replicate the above in ES6 modules by using the named exports syntax. Instead of assigning to `module.exports` like with CommonJS, in ES6 you can declare bindings you want to `export`. Note that the code below cannot be refactored to extract the variable declarations into standalone statements and then just `export foo`, as that'd be a syntax error. Here again, we see how ES6 modules favor static analysis by being rigid in how the declarative module system API works.

```js
export var foo = 'bar'
export var baz = 'ponyfoo'
```

It's important to keep in mind that we are exporting _bindings_.

## Bindings, Not Values

An important point to make is that ES6 modules export bindings, not values or references. That means that a `foo` variable you export would be bound into the `foo` variable on the module, and its value would be subject to changes made to `foo`. I'd advise against changing the public interface of a module after it has initially loaded, though.

If you had an `./a` module like the one found below, the `foo` export would be bound to `'bar'` for 500ms and then change into `'baz'`.

```js
export var foo = 'bar'
setTimeout(() => foo = 'baz', 500)
```

Besides a "default" binding and individual bindings, you could also export lists of bindings.

## Exporting Lists

As seen in the snippet below, ES6 modules let you `export` _lists_ of named top-level members.

```js
var foo = 'ponyfoo'
var bar = 'baz'
export <mark>{ foo, bar }</mark>
```

If you'd like to export something with a different name, you can use the `export { foo as bar }` syntax, as shown below.

```js
export { foo <mark>as ponyfoo</mark> }
```

You could also specify `as default` when using the named member list `export` declaration flavor. The code below is the same as doing `export default foo` and `export bar` afterwards -- but in a single statement.

```js
export { foo <mark>as default</mark>, bar }
```

There's many benefits to using only `export default`, and only at the bottom of your module files.

# Best Practices and `export`

Having the ability to define named exports, exporting a list with aliases and whatnot, and also exposing a a _"default"_ `export` will mostly introduce confusion, and _for the most part_ I'd encourage you to use `export default` -- and to do that at the end of your module files. You could just call your API object `api` or name it after the module itself.

```js
var <mark>api</mark> = {
  foo: 'bar',
  baz: 'ponyfoo'
}
export default <mark>api</mark>
```

<mark>One</mark>, the exported interface of a module becomes immediately obvious. Instead of having to crawl around the module and put the pieces together to figure out the API, you just scroll to the end. Having a clearly defined place where your API is exported also makes it easier to reason about the methods and properties your modules export.

<mark>Two</mark>, you don't introduce confusion as to whether `export default` or a named export -- or a list of named exports _(or a list of named exports with aliases...)_ -- should be used in any given module. There's a guideline now -- just use `export default` everywhere and be done with it.

<mark>Three</mark>, consistency. In the CommonJS world _it is usual_ for us to export a single method from a module, and that's it. Doing so with named exports is impossible as you'd effectively be exposing an object with the method in it, unless you were using the `as default` decorator in the `export` list flavor. The `export default` approach is more versatile because it allows you to `export` just one thing.

<mark>Four</mark>, -- and this is really a reduction of points made earlier -- the `export default` statement at the bottom of a module makes it immediately obvious what the exported API is, what its methods are, and generally easy for the module's consumer to `import` its API. When paired with the convention of _always using `export default` and always doing it at the end of your modules_, you'll note using the ES6 module system to be painless.

Now that we've covered the `export` API and its caveats, let's jump over to `import` statements.

# `import`

These statements are the counterpart of `export`, and they can be used to load a module from another one -- first and foremost. The way modules are loaded is _implementation-specific_, and at the moment no browsers implement module loading. This way you can write spec-compliant ES6 code today while smart people figure out how to deal with module loading in browsers. Transpilers like Babel are able to concatenate modules with the aid of a module system like CommonJS. That means `import` statements in Babel follow _mostly_ the same semantics as `require` statements in CommonJS.

Let's take [`lodash`][2] as an example for a minute. The following statement simply loads the Lodash module from our module. It doesn't create any variables, though. It **will execute** any code in the top level of the `lodash` module, though.

```js
<mark>import</mark> 'lodash'
```

Before going into importing bindings, let's also make a note of the fact that `import` statements, -- much like with `export` -- are only allowed in the top level of your module definitions. This can help transpilers implement their module loading capabilities, as well as help other static analysis tools parse your codebase.

## Importing Default Exports

In CommonJS you'd `import` something using a `require` statement, like so.

```js
var _ = require('lodash')
```

To import the default exported binding from an ES6 module, you just have to pick a name for it. The syntax is a bit different than declaring a variable because you're importing a _binding_, and also to make it easier on static analysis tools.

```js
import <mark>_</mark> from 'lodash'
```

You could also import named exports and alias them.

## Importing Named Exports

The syntax here is very similar to the one we just used for default exports, you just add some braces and pick any named exports you want. Note that this syntax is similar to the [destructuring assignment syntax][3], but also bit different.

```js
import <mark>{map, reduce}</mark> from 'lodash'
```

Another way in which it differs from destructuring is that you could use aliases to rename imported bindings. You can mix and match aliased and non-aliased named exports as you see fit.

```js
import {cloneDeep <mark>as clone</mark>, map} from 'lodash'
```

You can also mix and match named exports and the default export. If you want it inside the brackets you'll have to use the `default` name, which you can alias; or you could also just mix the default import side-by-side with the named imports list.

```js
import {<mark>default</mark>, map} from 'lodash'
import {default <mark>as _</mark>, map} from 'lodash'
import <mark>_</mark>, {map} from 'lodash'
```

Lastly, there's the `import *` flavor.

## `import` All The Things

You could also import the namespace object for a module. Instead of importing the named exports or the default value, it imports all the things. Note that the `import *` syntax must be followed by an alias where all the bindings will be placed. If there was a default export, it'll be placed in `alias.default`.

```js
import * as _ from 'lodash'
```

That's about it!

# Conclusions

Note that you can use ES6 modules today through the Babel compiler while leveraging CommonJS modules. What's great about that is that you can actually interoperate between CommonJS and ES6 modules. That means that even if you `import` a module that's written in CommonJS it'll actually work.

The ES6 module system looks great, and it's one of the most important things that had been missing from JavaScript. I'm hoping they come up with a finalized module loading API and browser implementations soon. The many ways you can `export` or `import` bindings from a module don't introduce as much versatility as they do added complexity for little gain, but time will tell whether all the extra API surface is as convenient as it is large.

[1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode "Strict Mode on MDN"
[2]: http://lodash.com/docs "Lodash is a JavaScript 'utility belt' library"
[3]: /articles/es6-destructuring-in-depth "ES6 JavaScript Destructuring in Depth on Pony Foo"
