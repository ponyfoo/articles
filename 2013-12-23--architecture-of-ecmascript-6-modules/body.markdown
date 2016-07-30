This document covers a number of use-cases covered by existing
module implementations in JavaScript, and how those use-cases
will be handled by ES6 modules.

It will also cover some additional use-cases unique to the ES6
module system.

## Terminology

For those unfamiliar with the current ES6 module proposal, here
is some terminology you should understand:

1. _module_: a unit of source code with optional _imports_ and
  _exports_.
2. _export_: a module can `export` a value with a name.
3. _imports_: a module can `import` a value exported by another
  module by its name.
4. _module instance object_: an instance of the `Module` constructor
  that represents a _module_. Its property names and values come
  from the _module_'s exports.
5. _Loader_: an object that defines how modules are fetched,
  translated, and compiled into a _module instance object_.
  Each JavaScript environment (the browser, node.js) defines
  a default Loader that defines the semantics for that environment.

## Imports and Exports

Let's start with the basic API of ES6 modules:

```javascript
// libs/string.js

var underscoreRegex1 = /([a-z\d])([A-Z]+)/g,
    underscoreRegex2 = /\-|\s+/g;

export function underscore(string) {
  return string.replace(underscoreRegex1, '$1_$2')
               .replace(underscoreRegex2, '_')
               .toLowerCase();
}

export function capitalize(string) {
  return string.charAt(0).toUpperCase() + string.substr(1);
}
```

```javascript
// app.js

import { capitalize } from "libs/string";

var app = {
  name: capitalize(document.title)
};

export app;
```

This illustrates the basic syntax of ES6 modules. A module can
export named values, and other modules can import those values.

## Avoiding Scope Pollution

When working with a module with a large number of exports, you
may want to avoid adding each of them as top-level names of
another module that wants to import it.

For example, consider an API like [Node.js fs module][1]. This
module has a large number of exports, like `rename`, `chown`,
`chmod`, `stat` and others. With the ES6 module API, it is
possible to bring in the module as a single top-level name
that contains all of the module's exports.

```javascript
import "fs" as fs;

fs.rename(oldPath, newPath, function(err) {
  // continue
});
```

[1]: http://nodejs.org/api/fs.html

## Concatenation

In the example above, the modules were loaded based on their
location on the file system. This is how the default Loader
for the browser will work.

For production applications, you will want to concatenate the
files on the file system into a single file. ES6 modules handle
this case by providing a literal way to define a module:

```js
module "libs/string" {
  var underscoreRegex1 = /([a-z\d])([A-Z]+)/g,
      underscoreRegex2 = /\-|\s+/g;

  export function underscore(string) {
    return string.replace(underscoreRegex1, '$1_$2')
                 .replace(underscoreRegex2, '_')
                 .toLowerCase();
  }

  export function capitalize(string) {
    return string.charAt(0).toUpperCase() + string.substr(1);
  }
}

module "app" {
  import { capitalize } from "libs/string";

  var app = {
    name: capitalize(document.title)
  };

  export app;
}
```

Modules defined using this syntax will be available to other
modules, and will not needed to be fetched through the Loader.

## Modules in Non-Default Locations

In web applications, while many modules may be concatenated into
a single file for production use, some modules, like jQuery,
may be loaded off of a CDN.

It is possible to override the default Loader hooks to specify
where to load a module from, but ES6 modules provide a simple
API for mapping modules to their physical location.

```javascript
System.ondemand({
  "https://ajax.googleapis.com/jquery/2.4/jquery.module.js": "jquery",
  "backbone.js": ["backbone/events", "backbone/model"]
});
```

The first line in the example specifies that the `jquery` module
can be found at `https://ajax.googleapis.com/jquery/2.4/jquery.module.js`.

The second line specifies that `backbone/events` and `backbone/model`
can both be found at `backbone.js`.

You can call `System.ondemand` as many times as you want, so
libraries can provide a snippet of code for people to use in
order to _import_ their libraries.

## The Compilation Pipeline

The next several sections deal with various use-cases involving
the compilation pipeline.

Here is a high-level overview of the process.

<img src="https://i.imgur.com/6yAVeq2.png">

The dotted line between **fetch** and **translate** reflects the fact
that process of retrieving the source is asynchronous.

### Stricter Mode (Linting)

Linting tools are a crucial part of a JavaScript developer's 
workflow, but they are currently used primarily via a compilation 
toolchain that presents errors in the terminal.

Using the Module Loader's translate hook, it is possible to
add additional static checks that are presented to the user as
`SyntaxError`s.

```javascript
import { JSHINT } from "jshint";
import { options } from "app/jshintrc"

System.translate = function(source, options) {
  var errors = JSHINT(source, options), messages = [options.actualAddress];

  if (errors) {
    errors.forEach(function(error) {
      var message = '';
      message += error.line + ':' + error.character + ', ';
      message += error.reason;
      messages.push(message);
    });

    throw new SyntaxError(messages.join("\n"));
  }

  return source;
};
```

If the linter returns errors, the translate hook raises a
`SyntaxError` and the Loader pipeline will stop, throwing the 
exception as if it was a true `SyntaxError`.

<img src="https://i.imgur.com/tbeB9TJ.png">

### Importing Compile-to-JavaScript Modules (CoffeeScript)

Increasingly, modules are written using languages that compile
to JavaScript.

The `translate` hook provides a way to translate source code
to JavaScript before it is loaded as a module.

```javascript
System.translate = function(source, options) {
  if (!options.path.match(/\.coffee$/)) { return; }

  return CoffeeScript.translate(source);
};
```

In this example, any modules ending in `.coffee` will be translated
from CoffeeScript to JavaScript, and the rest of the pipeline will
just see the compiled JavaScript.

<img src="https://i.imgur.com/GIAQldl.png">

### Verification and Translation

Some other compilers, like [TypeScript][2] and [restrict mode][3]
perform both compile-time verification and source translation.

The above techniques could be combined to produce seamless in-browser
support for such libraries.

[2]: http://typescript.codeplex.com/
[3]: http://restrictmode.org/

### Using Existing Libraries as Modules

The existing jQuery library is distributed as a library
that "exports" the `jQuery` name onto the global object.

It should be possible to import existing libraries without
having to modify the original source, like this:

```javascript
import { jQuery } from "jquery";

jQuery(function($) {
  $(".ui-button").button();
});
```

The final hook in the process, **link** can be  used to manually
process a source file into a _Module_ object.

In this case, we could configure the Loader to extract all 
properties written to `window`.

```javascript
function extractExports(loader, original) {
  source =
    `var exports = {};
    (function(window) {
      ${source};
    })(exports);
    exports;`

  return loader.eval(source);
}

System.link = function(source, options) {
  if (options.metadata.type === 'legacy') {
    return new Module(extractExports(this, source));
  }

  // returning undefined will result in the normal
  // parsing and registration behavior
}
```

In order to make it easy for the **link** hook to decide whether
it should use custom linking logic, the `resolve` hook can provide 
metadata for the module that will be passed to the following hooks.

In this case, you can keep a list of which modules are "legacy" and
populate the metadata with that information in `resolve`:

```javascript
var legacy = ["jquery", "backbone", "underscore"];

System.resolve = function(path, options) {
  if (legacy.indexOf(path) > -1) {
    return { name: path, metadata: { type: 'legacy' } };
  } else {
    return { name: path, metadata: { type: 'es6' } };
  }
}
```

<img src="https://i.imgur.com/y6Jjk0o.png">

### Importing AMD Modules from ES6 Modules

Similarly, you may want to import an AMD module's exports in an ES6
module.

Consider a simple AMD module for the string formatting example above:

```javascript
// libs/string.js

define(['exports'], function(exports) {
  var underscoreRegex1 = /([a-z\d])([A-Z]+)/g,
      underscoreRegex2 = /\-|\s+/g;

  exports.underscore = function(string) {
    return string.replace(underscoreRegex1, '$1_$2')
                 .replace(underscoreRegex2, '_')
                 .toLowerCase();
  }

  exports.capitalize = function(string) {
    return string.charAt(0).toUpperCase() + string.substr(1);
  }
});
```

To assimilate this module, you could use a similar technique to the
one we used above for jQuery:

```javascript
var amd = ["string-utils"];

// Resolve 
System.resolve = function(path, options) {
  if (amd.indexOf(path) > -1) {
    return { name: path, metadata: { type: 'amd' } };
  } else {
    return { name: path, metadata: { type: 'es6' } };
  }
};

function extractAMDExports(loader, source) {
  var loader = new Loader();
  loader.eval(`
    var module;
    var define = function(deps, callback) {
      module = { deps: deps, callback: callback };
    };
    ${source};
    module;
  `);

  // Assume synchronously available dependencies. See below
  // for a discussion of async dependencies.
  var exports = {};
  var deps = module.deps.map(function(name) {
    // AMD uses a special dependency named `exports` to
    // collect exports.
    if (name === 'exports') { return exports; }
    else { return loader.get(name); }
  });

  callback(deps);
  return exports;
}

System.link = function(source, options) {
  if (options.metadata.type === 'amd') {
    return new Module(extractAMDExports(this, source));
  }
}
```

To be clear, the particular implementation here is simple, and a real
approach to AMD assimilation would be more complicated. This should 
provide some idea of what such an approach would look like.

<img src="https://i.imgur.com/gl9TMbU.png">

### Importing Node Modules from ES6 Modules

The approach to importing node modules from ES6 modules is similar. 
Consider a node version of the above module:

```javascript
var underscoreRegex1 = /([a-z\d])([A-Z]+)/g,
    underscoreRegex2 = /\-|\s+/g;

exports.underscore = function(string) {
  return string.replace(underscoreRegex1, '$1_$2')
               .replace(underscoreRegex2, '_')
               .toLowerCase();
}

exports.capitalize = function(string) {
  return string.charAt(0).toUpperCase() + string.substr(1);
}
```

You'd override the hooks in a similar way:

```javascript
var node = ["string-utils"];

// Resolve 
System.resolve = function(path, options) {
  if (node.indexOf(path) > -1) {
    return { name: path, metadata: { type: 'node' } };
  } else {
    return { name: path, metadata: { type: 'es6' } };
  }
};

function extractNodeExports(loader, source) {
  var loader = new Loader();
  return loader.eval(`
    var exports = {};
    ${source};
    exports;
  `);
}

System.link = function(source, options) {
  if (options.metadata.type === 'node') {
    return new Module(extractNodeExports(this, source));
  }
}
```

### Importing From Multiple Non-ES6 Modules

To import from all three of these external module systems together,
you would write a resolve hook that would store off the type of 
module in the context, and then use that information to evaluate
the source appropriately in the `link` hook.

To make this process easier, a JavaScript library like `require.js`, 
built for the ES6 loader, could provide conveniences for registering
the type of external modules and assimilation code for `link`.

### Import a "Single Export" From a Non-ES6 Module

Some external module systems support modules that have a single 
export, rather than a number of named exports.

The techniques described above could be used to register that
single export under a conventionally known name.

Consider the following "single export" module using node-style 
modules:

```javascript
// string-utils/capitalize.js

module.exports = function(string) {
  return string.charAt(0).toUpperCase() + string.substr(1);
}
```

In order to support using this module in an ES6 module, a loader
can create a conventional name for the export that ES6 modules can
import.

In this example, we will name the export `exports` for consistency
with existing node practice. Once we have done this, ES6 modules will
be able to import the module:

```javascript
// app.js

import { exports: capitalize } from "string-utils/capitalize";

console.log(capitalize("hello")) // "Hello"
```

Here, we are renaming the conventionally named `exports` to
`capitalize`.

In order to achieve this, we will augment the earlier node 
assimilation code to handle `module.exports =` semantics.

```javascript
function extractNodeExports(loader, source) {
  var loader = new Loader();
  var exports = loader.eval(`
    var module = {};
    var exports = {};
    ${source};
    { single: module.exports, named: exports };
  `);

  if (exports.single !== undefined) {
    return { exports: exports.single }
  } else {
    return exports.named;
  }
}

System.link = function(source, options) {
  if (options.metadata.type === 'node') {
    return new Module(extractNodeExports(this, source));
  }
}
```

A similar approach could be used to allow assimilated AMD modules
to have a "single export".

### Importing an ES6 Module From a Node Module

When using a node module, we would want to be able to import any
other module, regardless of the source.

One major benefit of the above approaches to importing non-ES6 
modules is that it means that the standard `System.get` will be
able to load them.

This means that it's easy to support `require` in a node module:
just alias it to `System.get`.

```javascript
function extractNodeExports(loader, source) {
  var loader = new Loader();
  var exports = loader.eval(`
    var module = {};
    var exports = {};
    var require = System.get;
    ${source};
    { single: module.exports, named: exports };
  `);

  if (exports.single !== undefined) {
    return { exports: exports.single }
  } else {
    return exports.named;
  }
}
```

### Importing an AMD Module With Asynchronous Dependencies

In the above examples, we assumed that all dependencies in external
modules are available synchronously, so we could use System.get in
the `link` hook.

AMD modules can have asynchronous dependencies that can be determined
without having to execute the module.

For this use-case, you can return (from `link`) a list of 
dependencies and a callback to call once the Loader has loaded the 
dependencies. The callback will receive the list of dependencies as 
parameters and must return a `Module` instance.


```javascript
var amd = ['string-utils'];

System.resolve = function(path, options) {
  if (amd.indexOf(path) !== -1) {
    options.metadata = { type: 'amd' };
  } else {
    options.metadata = { type: 'es6' };
  }
};

System.link = function(source, options) {
  if (options.metadata.type !== 'amd') { return; }

  var loader = new Loader();
  var [ imports, factory ] = loader.eval(`
    var dependencies, factory;
    function define(dependencies, factory) {
      imports = dependencies;
      factory = factory;
    }
    ${source};
    [ imports, factory ];
  `);

  var exportsPosition = imports.indexOf('exports');
  imports.splice(exportsPosition, 1);

  function execute(...args) {
    var exports = {};
    args.splice(exportsPosition, 0, [exports]);
    factory(...args);
    return new Module(exports);
  }

  return { imports: imports, execute: execute };
};
```

Returning the imports and a callback from `link` allows the `link`
hook to participate in the same two-phase loading process of ES6
modules, but using the AMD definition to separate the phases instead
of ES6 syntax.

<img src="https://i.imgur.com/AO3mEIm.png">

### Importing a Node Module By Processing `require`s

Because node modules use a dynamic expression for imports, there is 
no perfectly reliable way to ensure that all dependencies are loaded 
before evaluating the module.

The approach used by _Browserify_ is to statically analyze the file 
first for `require` statements and use them as the dependencies. The
_AMD CommonJS wrapper_ uses a similar approach.

The `link` hook could be used to analyze Node-style packages for `require` lines, and return them as `imports`.

By the time the `execute` callback was called, all modules would be
synchronously available, and aliasing `require` to `System.get` 
would continue to work.

```javascript
import { processImports } from "browserify";

System.link = function(source, options) {
  var imports = processImports(source);

  function execute() {
    return new Module(extractNodeExports(source));
  }

  return { imports: imports, execute: execute};
};
```

Of course, this means only works as long as no `requires` are used 
with dynamic expressions, in a conditional, or in a `try/catch`, but
those are already limitations of systems like _Browserify_.

<img src="https://i.imgur.com/7EMzTG2.png">

### Interoperability in General

Let's review the overall strategy used for assimilating non-ES6 
module definitions:

* Non-ES6 modules can be loaded through the Loader by overriding the
  `resolve` and `link` hooks.
* Non-ES6 modules can asynchronously load other modules by 
  return imports from `link` and synchronously through
  `System.get`.

This means that all module systems can freely interoperate, using
the Loader as an intermediary.

For example, if an AMD module (say, 'app'), depended on a Node-style 
module (say, 'string-utils'):

1. When loading `app`, the `link` hook would return
   `{ imports: ['string-utils'], execute: execute }`.
2. This would cause the Loader to attempt to load `'string-utils'`,
   before it would call back the provided `execute` callback.
3. The Loader would fetch `string-utils` and evaluate it using the
   Node-style `link` hook.
4. Once this is done, the provided `execute` callback would run, 
   receiving the `string-utils` Module as a parameter.
5. The `execute` callback would then return a Module.

This is just an illustrative example; any combination of module 
systems could freely interoperate through the Loader.

### A Note on "Single Export" Interoperability

Many of the existing module systems support mechanisms for exporting
a single value instead of a number of named values from a module.

At the current time, ES6 modules do not provide explicit support for
this feature, but it can be emulated using the Loader. One specific
strategy would be to export the single value as a well-known name
(for example, `exports`).

Let's take a look at how a Loader could support a Node-style module
using `require` to import the "single export" of another Node-style
module.

This same approach would support interoperability between module
systems that support importing and exporting of single values.

We'll need to enhance the previous solution we provided for this
scenario:

```javascript
var isSingle = new Symbol();

function extractNodeExports(loader, source) {
  var loader = new Loader();
  var exports = loader.eval(`
    var module = {};
    var exports = {};
    var require = System.get;
    ${source};
    { single: module.exports, named: exports };
  `);

  if (exports.single !== undefined) {
    return { exports: exports.single, [isSingle]: true };
  } else {
    return exports.named;
  }
}

System.link = function(source, options) {
  if (options.metadata.type === 'node') {
    return new Module(extractNodeExports(this, source));
  }
}
```

Here, we create a new unique Symbol that we will use to tag a module
as containing a single export. This will avoid conflicts with 
Node-style modules that export the name `exports` explicitly.

Next, we will need to enhance the code that we have been using for 
Node-style `require`. Until now, we have simply aliased it to
`System.get`. Now, we will check for the `isSingle` symbol and give
it special treatment in that case.

```javascript
// this assumes that the `isSingle` Symbol is in scope
var require = function(name) {
  var module = System.get(name);
  if (module[isSingle]) {
    return module.exports;
  } else {
    return module;
  }
}
```

This same approach, using a shared `isSingle` symbol, could be used
to support interoperability between AMD and Node single exports.

As described earlier, ES6 modules would use
`import { exports: underscore } from 'string-utils/underscore'`.

## Configuration of Existing Loaders

The `requirejs` loader has a number of useful configuration options 
that its users can use to control the loader.

This section covers a sampling of those options and how they map onto
the semantics of the ES6 Loader. In general, the compilation pipeline
provides hooks that can be used to implement these configuration 
options.

### Base URL

The `requirejs` loader allows the user to configure a base URL for
resolving relative paths.

In the default browser loader, the base URL will default to the 
page's base URL. The default `System.resolve` will prefix that base
URL and append `.js` to the end of the module name (if not already
present).

The browser's default Loader (`window.System`) will also include a
`baseURL` configuration option that controls the base URL for its
implementation of `resolve`.

JavaScript code could also configure the Loader's resolve hook to provide any policy they like:

```javascript
var resolve = System.resolve;

System.resolve = function(name, ...args) {
  if (name.match(/fun/)) {
    return `/assets/javascripts/${name}.js"
  }
  return resolve(name, ...args);
};
```

### URL Arguments

Similarly, the `requirejs` loader allows the specification of 
additional URL arguments. This could also be handled by overriding
the `resolve` hook.

```javascript
var resolve = System.resolve;

System.resolve = function(...args) {
  return resolve(name, ...args) + "?bust=" + (new Date().getTime());
};
```

### Timeouts

The `requirejs` loader allows the specification of a timeout before
rejecting the request.

With the ES6 Loader, the `fetch` hook can be overridden to reject 
the fetch after some time has passed.

```javascript
var fetch = System.fetch;

System.fetch = function(url, options) {
  setTimeout(function() {
    options.reject("Timeout");
  }, 5000);

  fetch(url, options);
};
```

### Support for Legacy Modules

The `requirejs` loader provides a mechanism for declaring how a
legacy module should be interpreted:

```javascript
requirejs.config({
  shim: {
    backbone: {
      deps: ['underscore', 'jquery']
      exports: 'Backbone'
    },
  }
});
```

The example above under **Using Existing Libraries as Modules** shows
one approach to this problem. That approach should work generically,
without having to list a specific export name.

The `link` hook provides a way to define dependencies for legacy
modules.

```javascript
var config = {
  backbone: {
    deps: ['underscore', 'jquery'],
    exports: ['Backbone']
  }
}

function executeCallback(source, exportNames) {
  System.eval(source);
  var exports = {};
  exportNames.forEach(function(name) {
    exports[name] = System.global[name]
  });
  return new Module(exports);
}

System.link = function(source, options) {
  if (!config[options.normalized]) { return; }

  var { deps, exports: exportNames } = config[options.normalized];

  if (moduleConfig) {
    return {
      imports: moduleConfig.deps,
      execute: executeCallback(source, exportNames);
    }
  }
};
```

## Referencing Modules in HTML

In Ember.js, Angular.js, and other contemporary frameworks, 
JavaScript objects are referenced in HTML templates:

```html
<!-- ember.js -->
{{#view App.FancyButton}}
<p>Fancy Button Contents</p>
{{/view}}
```

Here, the app is asking Ember.js to render some HTML defined
in an `App.FancyButton` constructor. Note that Ember encourages
the use of a global namespace for coordination between JavaScript
and HTML templates.

```html
<!-- angular -->
<button fancy-button>
  <p>Fancy Button Contents</p>
</button>
```

Here, the app is asking Angular.js to replace the `<button>`
with some content defined in a globally registered `fancy-button`
directive.

Both Angular and Ember both use globally registered names to
define _controller_ objects to attach to parts of the HTML
controlled by the framework.

```html
<!-- ember -->
{{control "fancy-button"}}
```

Here, the app is asking Ember.js to render some HTML defined
in an `App.FancyButtonView` and use an instance of the
`App.FancyButtonController` as its controller. Again, Ember
is relying on a globally rooted namespace for coordination.

```html
<!-- angular -->
<div ng-controller="TodoCtrl">
  <span>{{remaining()}} of {{todos.length}} remaining</span>
</div>
```

Here, the app is asking Angular to use a globally rooted
object called `TodoCtrl` as the controller for this part
of the HTML. In Angular, this controller is used to control
the scope for data-bound content nested inside of its element.

To handle the kind of situation where a module is referenced
by a String and needs to be looked up dynamically, ES6
modules provide an API for looking up a module at runtime.

```javascript
System.get('controllers/fancy-button');
```

Systems like Ember or Angular could use this API to allow their
users to reference a module's exports in HTML.

In the first Ember example, instead of referencing a globally
rooted constructor, the HTML would reference a module name:

```html
<!-- ember.js -->
{{#view views/fancy-button}}
<p>Fancy Button Contents</p>
{{/view}}
```

And the module would look like:

```javascript
// views/fancy-button.js
import { View } from "ember";

export let view = View.extend({
  // contents
});
```

The second Angular example could be rewritten as:

```html
<!-- angular -->
<div ng-controller="controllers/todo">
  <span>{{remaining()}} of {{todos.length}} remaining</span>
</div>
```

And the JavaScript:

```javascript
// controllers/todo.js

export function Controller($scope) {
  // contents
}
```

The general pattern is to switch from globally rooted namespaces
to named, registered modules. `System.get` provides a way to
dynamically look up already loaded modules.

## Creating Modules from HTML

The new Web Components specification provides a way to create
a JavaScript constructor through HTML:

```html
<element extends="button" name="x-fancybutton" constructor="FancyButton">
  <script>
    FancyButton.prototype.razzle = function () {
    };
    FancyButton.prototype.dazzle = function () {
    };
  </script>
</element>
```

```javascript
// app.js

var b = new FancyButton();
b.textContent = "Show time";
document.body.appendChild(b);
b.addEventListener("click", function (event) {
    event.target.dazzle();
});
b.razzle();
```

Here, the `<element>` tag is creating a globally rooted
name for the constructor.

The specifics will probably vary in practice, but something
like this could work:

```
<element extends="button" name="x-fancybutton" module="web/x-fancybutton">
  <script>
  // automatically imports Element from web/x-fancybutton
  Element.prototype.razzle = function () {
  };
  Element.prototype.dazzle = function () {
  };
  </script>
</element>
```

```javascript
// app.js

import { Element: FancyButton } from "web/x-fancybutton"

var b = new FancyButton();
b.textContent = "Show time";
document.body.appendChild(b);
b.addEventListener("click", function (event) {
    event.target.dazzle();
});
b.razzle();
```

> Originally [posted by wycats as a gist on GitHub][1], but I felt it deserved to get more public attention. Resources describing ES6 modules are scarce, and **rarely this detailed**. Head over to the gist for an interesting discussion on the state of ES6 modules.

Related links:

- [ECMAScript 6 modules: the future is now][3]
- [ECMAScript 6 modules in future browsers][4]
- [Traceur Compiler][5]
- [`grunt-traceur`][6]
- [ES6 Compatibility Table][7]
- [ES6 Code Samples in my upcoming JavaScript Application Design book][8]

If you want to toy around with the syntax, [ModuleLoader/es6-module-loader][2] is the most up-to-date polyfill I could find. Believe me, _I looked_.

  [1]: https://gist.github.com/wycats/51c96e3adcdb3a68cbc3 "wycats/modules.md on gist"
  [2]: https://github.com/ModuleLoader/es6-module-loader "ModuleLoader/es6-module-loader on GitHub"
  [3]:http://www.2ality.com/2013/07/es6-modules.html "ECMAScript 6 modules: the future is now"
  [4]: http://www.2ality.com/2013/11/es6-modules-browsers.html "ECMAScript 6 modules in future browsers"
  [5]: https://github.com/google/traceur-compiler "Traceur Compiler"
  [6]: https://github.com/aaronfrost/grunt-traceur "grunt-traceur on GitHub"
  [7]: http://kangax.github.io/es5-compat-table/es6/ "ES6 Compatibility Table"
  [8]: https://github.com/bevacqua/buildfirst/tree/latest/ch05/17_harmony-traceur "Harmony Samples using Google Traceur Compiler"
