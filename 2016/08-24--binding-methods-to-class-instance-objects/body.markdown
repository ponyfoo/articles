# Caveman Mode

In this scenario, we just bind methods by hand on the constructor.

```js
class Logger {
  constructor () {
    this.printName = this.printName.bind(this);
  }

  printName (name = 'there') {
    this.print(`Hello ${name}`);
  }

  print (text) {
    console.log(text); 
  }
}
```

This is the least elegant solution, but it works. Drawbacks include having to keep track of which methods use `this` and need to be bound, or ensuring every method is bound, remembering to `.bind` new methods as they are added, and removing `.bind` statements for methods that are removed. Benefits include being explicit, and having no extra code involved.

# Auto Bind

A similar but less painful approach is using a module that takes care of this on our behalf. Sindre's [`auto-bind`][autobind] goes through an object's methods and binds them to itself.

```js
class Logger {
  constructor () {
    autoBind(this);
  }

  printName (name = 'there') {
    this.print(`Hello ${name}`);
  }

  print (text) {
    console.log(text); 
  }
}
```

This approach works well for classes, although we can't escape the need for a `constructor`. The advantage is that we don't have to keep track of every single method by name when binding them. At the same time, if we're dealing with objects rather than classes, we need to ensure that `autoBind` gets called on the object after every method has been assigned to the object, or else some methods will be left unbound. Any methods added after `autoBind` gets called are unbound, and this means that in some situations `autoBind` is an even worse option than manually calling `.bind` on every method. 

# Proxies

A [`Proxy` object][proxies] could be used to intercept `get` operations, returning methods bound to the `logger`. Below we have a `selfish` function which takes an object and returns a proxy for that object. Any methods accessed through the proxy will be automatically bound to the object. A `WeakMap` is used to ensure we only bind the methods once, so that equality in `proxy.fn === proxy.fn` is preserved.

```js
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    get (target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}
const logger = selfish(new Logger());
```

The benefit of taking this approach is that methods on the `target` object will be bound regardless of whether they were added before or after the proxy object was created. The obvious drawback is that `Proxy` support is meager even when transpilers are taken into account.

Even if proxies were generally available, this solution would also be far from ideal. The difference lies in that libraries could potentially implement something like `selfish` so that components you hand over to the library would follow these semantics without you having to do anything. That said, the only real solution lies in the language moving forward, adding semantics for classes that have every method bound to itself by default.

We're definitely better off than back when we had to write `Logger.prototype.print` _-- an implementation detail artifact that never made a lot of sense --_ and classes are a result of observing a pattern that was being repeated consistently, over time, and developing a solution. A private scope for classes where you can declare functions scoped to that class that aren't class methods _(but are available to every method)_ and other variables scoped to the class would be a huge step forward for classes.

Having bind semantics that are a bit less verbose, tedious, error-prone, or complicated, is mostly a matter of time. And when it comes to JavaScript, we all know that time flows faster. ⏳

> What's your preferred way of ensuring methods are bound to their host object? What native JavaScript semantics would you propose for objects where every method is bound to their host?

[autobind]: https://github.com/sindresorhus/auto-bind "sindresorhus/auto-bind on GitHub"
[proxies]: /articles/es6-proxies-in-depth "ES6 Proxies in Depth on Pony Foo"
