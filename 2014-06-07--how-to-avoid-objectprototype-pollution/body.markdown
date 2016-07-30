Really it's not as complicated as it sounds. In Node, it involves **just 5 lines of code**. The [`vm` module][1] allows you to run code in a new execution context, meaning you get a brand new `Array.prototype`. Turns out, it's quite simple to grab a reference to any of that context's globals, and run with it. In this case, I'll be stealing the `Array` global.

```js
var vm = require('vm');

function poser () {
  var sandbox = {};
  vm.runInNewContext('stolen=Array;', sandbox, 'poser.vm');
  return sandbox.stolen;
}
```

Now every time I run `poser()` I'll get a brand new `Array` that I can extend at will, without affecting the `Array` everyone else uses. Modularity much? Of course, this isn't just limited to `Array`. You could do this with `Object`, `Function`, or just about anything that is accessible from the global namespace.

> You may be thinking, _"well, the browser doesn't have no stinking `vm` module!"_. It's a bit trickier, but it can be done too.

In the browser, the `<iframe>` tag is the go-to sandbox-provider. It provides an accessible execution context, and we can use that to steal their coveted prototypes away. Running a tiny bit of global-stealing JavaScript will do just fine. I'm only hiding the `<iframe>` rather than removing it, because I suspect removing it would make older browsers think it's okay to garbage collect the execution context for that `<iframe>`.

```js
var frames = global.frames;
var key = 'stolen, but no problem, just a poser';

function poser () {
  var iframe = document.createElement('iframe');
  var altdom;
  var stolen;

  iframe.style.display = 'none';
  document.body.appendChild(iframe);

  altdom = frames[frames.length - 1].document;
  altdom.write('<script>parent["' + key + '"]=Array;<\/script>');

  stolen = global[key];
  delete global[key]; // pollution-free environment!

  return stolen;
}
```

You'll have to **momentarily pollute the global namespace**, so that the parent frame can access the out-of-context variable, but you can safely vanquish the global afterwards. Using an inspired global variable name, or checking that the property didn't exist before, is enough. The only issue that I can think of is security mad men doing things like [`Object.freeze(window)`][2].

I'm **curious about the performance implications** in doing this, any comments on that will be thoroughly appreciated!

I, for one, will definitely be using [`poser`][3], and I'll make sure to comment on whether it's reasonable to go down that road.

[1]: http://nodejs.org/api/vm.html "Node.js API Documentation"
[2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze "Object.freeze - MDN"
[3]: https://github.com/bevacqua/poser "poser on GitHub"
