# RequireJS, AngularJS, and Dependency Injection

This is a problem we've hardly had to think about ever since the advent of module systems like RequireJS or the dependency injection mechanism in AngularJS, both of which allowed us to explicitly name the dependencies of each module.

The following example shows we might define the `mathlib/sum.js` library using RequireJS's `define` function, which was added to the global scope. The returned value from the `define` callback is then used as the public interface for our module.

```js
define(function() {
  return sum

  function sum(...values) {
    return values.reduce((a, b) => a + b, 0)
  }
})
```

We could then have a `mathlib.js` module which aggregates all functionality we wanted to include in our library. In our case, it's just `mathlib/sum`, but we could list as many dependencies as we wanted in the same way. We'd list each dependency using their paths in an array, and we'd get their public interfaces as parameters passed into our callback, in the same order.

```js
define(['mathlib/sum'], function(sum) {
  return { sum }
})
```

Now that we've defined a library, we can consume it using `require`. Notice how the dependency chain is resolved for us in the snippet below.

```js
require(['mathlib'], function(mathlib) {
  mathlib.sum(1, 2, 3)
  // <- 6
})
```

This is the upside in RequireJS and its inherent dependency tree. Regardless of whether our application contained a hundred or thousands of modules, RequireJS would resolve the dependency tree without the need for a carefully maintained list. Given we've listed dependencies exactly where they were needed, we've eliminated the necessity for a long list of every component and how they're related to one another, as well as the error-prone process of maintaining such a list. Eliminating such a large source of complexity is merely a side-effect, but not the main benefit.

This explicitness in dependency declaration, at a module level, made it obvious how a component was related to other parts of the application. That explicitness in turn fostered a greater degree of modularity, something that was ineffective before because of how hard it was to follow dependency chains.

RequireJS wasn't without problems. The entire pattern revolved around its ability to asynchronously load modules, which was ill-advised for production deployments due to how poorly it performed. Using the asynchronous loading mechanism, you issued hundreds of networks requests in a waterfall fashion before much of your code was executed. A different tool would have to be used to optimize builds for production. Then there was the verbosity factor, where you'd end up with long lists of dependencies, a RequireJS function call, and the callback for your module. On that note, there were quite a few different RequireJS functions and several ways of invoking those functions, complicating its use. The API wasn't the most intuitive, because there were so many ways of doing the same thing: declaring a module with dependencies.

The dependency injection system in AngularJS suffered from many of the same problems. It was an elegant solution at the time, relying on clever string parsing to avoid the dependency array, using function parameter names to resolve dependencies instead. This mechanism was incompatible with minifiers, which would rename parameters to single characters and thus break the injector.

Later in the lifetime of AngularJS v1, a build task was introduced that would transform code like the following:

```js
module.factory('calculator', function(mathlib) {
  // …
})
```

Into the format in the following bit of code, which was minification-safe because it included the explicit dependency list.

```js
module.factory('calculator', ['mathlib', function(mathlib) {
  // …
}])
```

Needless to say, the delay in introducing this little-known build tool, combined with the over-engineered aspect of having an extra build step to unbreak something that shouldn't have been broken, discouraged the use of a pattern that carried such a negligible benefit anyway. Developers mostly chose to stick with the familiar RequireJS-like hardcoded dependency array format.

# Node.js and the Advent of CommonJS

Among the many innovations hailed by Node.js, one was the CommonJS module system -- or CJS for short. Taking advantage of the fact that Node.js programs had access to the file system, the CommonJS standard is more in line with traditional module loading mechanisms. In CommonJS, each file is a module with its own scope and context. Dependencies are loaded using a synchronous `require` function that can be dynamically invoked at any time in the lifecycle of a module, as illustrated in the next snippet.

```js
const mathlib = require('./mathlib')
```

Much like RequireJS and AngularJS, CommonJS dependencies are also referred to by a pathname. The main difference is that the boilerplate function and dependency array are now both gone, and the interface from a module could be assigned to a variable binding, or used anywhere a JavaScript expression could be used.

Unlike RequireJS or AngularJS, CommonJS was rather strict. In RequireJS and AngularJS you could have many dynamically-defined modules per file, whereas CommonJS had a one-to-one mapping between files and modules. At the same time, RequireJS had several ways of declaring a module and AngularJS had several kinds of factories, services, providers and so on -- besides the fact that its dependency injection mechanism was tightly coupled to the AngularJS framework itself. CommonJS, in contrast, had a single way of declaring modules. Any JavaScript file was a module, calling `require` would load dependencies, and anything assigned to `module.exports` was its interface. This enabled better tooling and code introspection -- making it easier for tools to learn the hierarchy of a CommonJS component system.

Eventually, Browserify was invented as way of bridging the gap between CommonJS modules for Node.js servers and the browser. Using the `browserify` command-line interface program and providing it with the path to an entry point module, one could combine an unthinkable amount of modules into a single browser-ready bundle. The killer feature of CommonJS, the npm package registry, was decisive in aiding its takeover of the module loading ecosystem.

Granted, npm wasn't limited to CommonJS modules or even JavaScript packages, but that was and still is by and large its primary use case. The prospect of having thousands of packages (now over half million and steadily growing) available in your web application at the press of a few fingertips, combined with the ability to reuse large portions of a system on both the Node.js web server and each client's web browser, was too much of a competitive advantage for the other systems to keep up.

# ES6, `import`, Babel, and Webpack

As ES6 became standardized in June of 2015, and with Babel transpiling ES6 into ES5 long before then, a new revolution was quickly approaching. The ES6 specification included a module system native to JavaScript, often referred to as ECMAScript Modules (ESM).

ESM is largely influenced by CJS and its predecessors, offering a static declarative API as well as a promise-based dynamic programmative API, as illustrated next.

```js
import mathlib from './mathlib'
import('./mathlib').then(mathlib => {
  // …
})
```

In ESM, too, every file is a module with its own scope and context. One major advantage in ESM over CJS is how ESM has -- and encourages -- a way of statically importing dependencies. Static imports vastly improve the introspection capabilities of module systems, given they can be analyzed statically and lexically extracted from the abstract syntax tree (AST) of each module in the system. Static imports in ESM are constrained to the topmost level of a module, further simplifying parsing and introspection.

In Node.js v8.5.0, ESM module support was introduced behind a flag. Most evergreen browsers also support ESM modules behind flags.

Webpack is a successor to Browserify that largely took over in the role of universal module bundler thanks to a broader set of features. Just like in the case of Babel and ES6, Webpack has long supported ESM with both its `import` and `export` statements as well as the dynamic `import()` function. It has made a particularly fruitful adoption of ESM, in no little parts thanks to the introduction of a "code-splitting" mechanism whereby it's able to partition an application into different bundles to improve performance on first load experiences.

Given how ESM is native to the language, -- as opposed to CJS -- it can be expected to completely overtake the module ecosystem in a few years time.
