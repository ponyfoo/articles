# Decorator Fundamentals

JavaScript decorators apply to classes and any statically-defined properties, such as those found on an object literal declaration or in a `class` declaration _-- even if they were `static`, or accessors like `get` and `set`._ The [proposal for JavaScript decorators][prop] defines a decorator as:

- an expression _(that's preceded by an `@` sign)_
- that evaluates to a function
- that takes the `target`, `name`, and decorator `descriptor` as arguments
- and _optionally_ returns a decorator `descriptor` to install on the `target` object

To better understand decorators, it may be better to look at an example using a plain JavaScript object first.

```js
const dog = {
  name: 'Doug',
  legs: 4
};
```

Typically, we would think of the `dog` object literal declaration above as an atom: when the assignment statement is executed, the object literal is assigned to `dog`. In order to better understand decorators, though, it may be more useful to think of the object literal as follows, using [`Object.defineProperties`][defpp] instead. The following piece of code is functionally equivalent to the previous one, albeit more verbose.

```js
const literal = <mark>{}</mark>;
const dog = Object.defineProperties(<mark>literal</mark>, {
  <mark>name</mark>: {
    value: 'Doug',
    writable: true,
    enumerable: true,
    configurable: true
  },
  <mark>legs</mark>: {
    value: 4,
    writable: true,
    enumerable: true,
    configurable: true
  }
});
```

> <sub>_Properties are writable, enumerable, and configurable by default._</sub>

Great. Let's now imagine that _-- for some reason --_ we want to add a decorator to the amount of legs so that it's no longer `writable`. Our decorator is declared as `@readonly`. This must precede the property declaration for `legs`, as that's the property declaration we want to modify. The `@` is mandatory, and `readonly` should be an arbitrary JavaScript expression that evaluates to a function.

```js
const dog = {
  name: 'Doug',
  <mark>@readonly</mark>
  legs: 4
};
```

Let's start by looking at how our code would look like using `Object.defineProperties`, now that we have a decorator. We moved the `legs` property descriptor into a variable, we're calling the `readonly` decorator by passing in the `literal` variable that will be eventually assigned to `dog`, the `'legs'` property name, and the original `legsDescriptor` property descriptor. The legs property will be defined using either the return value from the `readonly` decorator, or a reference to the original property descriptor.

```js
const literal = {};
const <mark>legsDescriptor</mark> = {
  value: 4,
  writable: true,
  enumerable: true,
  configurable: true
};
const dog = Object.defineProperties(literal, {
  name: {
    value: 'Doug',
    writable: true,
    enumerable: true,
    configurable: true
  },
  legs: <mark>readonly(literal, 'legs', legsDescriptor) || </mark>legsDescriptor
});
```

Understanding how to write the `readonly` decorator so that the `legs` property becomes read-only should be trivial, and is demonstrated below: _we just modify the original `descriptor` so that the property isn't `writable`_.

```js
function readonly (target, prop, descriptor) {
  descriptor.<mark>writable = false</mark>;
}
```

Given that the original descriptor is used if nothing is returned, we didn't necessarily need to `return` a value.

# Stacking Decorators and a Warning about Immutability

With all the fluff around immutability you may be tempted to `return` a new property descriptor, as to not modify the original descriptor. While well-intentioned, this may have **an undesired effect**. It is possible to decorate the same property or `class` several times. 

If any of the decorators in the following piece of code returned an entirely new `descriptor` without taking into account the `descriptor` supplied to the decorator function, you'd effectively lose all the decoration that took place before returning a different descriptor.

```js
const dog = {
  @readonly
  @nonenumerable
  @doubledValue
  legs: 4
};
```

Thus, we should be careful to write decorators that take into account the supplied `descriptor`: modify it directly or create one that's based on the `descriptor` that's provided to you.

# A Little Back Story: RunUO and Attributes in C#

A long time ago, in a galaxy far far away, I was getting acquainted with C# -- without even knowing -- by way of a [Ultima Online][uo] server emulator written in open-source C# code: [RunUO][ruo]. RunUO was one of the most beautiful codebases I've ever worked with, and it was written in C# to boot.

They distributed the server software as an executable and a series of `.cs` files. The `runuo` executable would compile those `.cs` scripts at runtime and dynamically mix them into the application. The result was that you didn't need the Visual Studio IDE _(nor `msbuild`)_, or really anything other than knowing just enough programming to edit one of the "scripts" written in `.cs` files. All of the above made RunUO a perfect learning environment for me.

I didn't know C# at the time. I didn't even know C# was a heavily used enterprise language. I just thought it was a beautiful language and a beautifully written codebase for the server-side of an MMORPG game I loved, so saying I was **eager to learn more** would be an understatement.

RunUO relied heavily in reflection. They made a significant effort to be customizable by players who were interested in changing a few details of the game _(such as how much damage a Dragon's fire breath inflicts)_, but not necessarily invested in programming itself. Good UX was a big part of their philosophy, and so you could create a new kind of Dragon just by copying one of the monster files, changing it to inherit from the `Dragon` class, and modifying a few properties to change its color, its damage output, etc.

Just as they made it easy to create new monsters, -- or "non-player characters" (NPC) in gaming slang -- they also relied in reflection to provide functionality to in-game administrators _("Game Masters")_. Game masters could run an in-game command and click on an item or a moster to visualize or change properties without ever leaving the game. Again, great UX.

![Properties for an item in RunUO][propgump]

> I used to go by "Kenko" and participate on the RunUO community. I guess this was one of my first adventures into the world of open-source, [back around 2006][duelp].

Not every property in a class is meant to be accessible in-game, though. The consequences of doing something like that could be catastrophic if unforeseen. RunUO had [a `CommandPropertyAttribute` decorator][commandpropdef] where you could specify the access level required to read and write properties. This decorator was [used extensively throughout the RunUO codebase][commandprops].

The `PlayerMobile` class, which governed how a player's character works, is a great place to look at these attributes. [`PlayerMobile` have several properties that are accessible in-game][pm] to administrators and moderators. Here are a couple of getters and setters, but only the first one has the `CommandProperty` attribute _-- making it in-game accessible to Game Masters._

```js
[CommandProperty( AccessLevel.GameMaster )]
public int Profession
{
  get{ return m_Profession; }
  set{ m_Profession = value; }
}

public int StepsTaken
{
  get{ return m_StepsTaken; }
  set{ m_StepsTaken = value; }
}
```

One interesting difference between C# attributes and JavaScript decorators is that reflection in C# allows us to pull all custom attributes from an object using [`MemberInfo#getCustomAttributes`][gca]. RunUO leverages that method to pull up information about each property that should be accessible in-game [when displaying the dialog][gcaruo] that lets an administrator view or modify an in-game object's properties.

# Marking "special" properties in JavaScript _-- or, defining protocols_

In JavaScript, there's no such thing _-- in this proposal draft, at least --_ to get the custom attributes on a property. That said, JavaScript is a highly dynamic language, and creating this sort of "labels" wouldn't be much of a hassle. Decorating a `dog` with a "command property" wouldn't be all that different from RunUO and C#.

```js
var dog = {
  @commandProperty('gm')
  legs: 4
};
```

The `commandProperty` function would have to be a little more sophisticated. Given that there is no reflection around decorators yet, we could use [a runtime-wide symbol][symbols] to define [a `Map`][maps] of special properties found on `target`. In the example shown above, `target` would be the `dog` object. Note that we aren't even touching the `descriptor` _(nor returning a different one)_. This is okay because we're more concerned with [the protocol][protocols] we've established than with implementation details for this specific property.

```js
function commandProperty (read, write) {
  return (target, prop, descriptor) => {
    const commandProperties = Symbol.for('commandProperties');
    if (!target[commandProperties]) {
      target[commandProperties] = new Map();
    }
    target[commandProperties].set(prop, {
      readLevel: read,
      writeLevel: write || read
    });
  };
}
```

The `dog` object could have as many command properties as neccessary, and each would be properly mapped behind a symbol property. To find out which special command properties any given object has, all we'd have to do is use the following one-liner, [spreading all special properties][spread] as `[key, value]` pairs into an array.

```js
[...target[Symbol.for('commandProperties')] || []]
// <- [['legs', { readLevel: 'gm', writeLevel: 'gm' }]]
```

You could then iterate over these special properties that are known to be changeable by a user who may not be as computer savvy. They may just want to change the amount of `legs` one given `dog` has. Instead of maintaining long lists of properties that can be modified, relying on some sort of heuristics bound to break from time to time, or using some sort of restrictive naming convention, decorators are the cleanliest way to implement a protocol such as this where we mark properties as special for some particular use case.

Granted, the pattern doesn't translate perfectly into JavaScript. Being a dynamic language, properties defined outside of an object literal would have to rely on a different methodology to mark them as special. I'll leave you to do the thinking for that one!

# ECMAScript Decorators Moving Forward

There are still moving parts in this proposal. For that reason, an "official" transpiler for decorators isn't available in Babel 6. You can learn more about the [thought process and current state of decorators][babelstate] in Babel's core modules on Phabricator _-- Babel's issue tracker._

Instead, you'll have to rely on [`babel-plugin-transform-decorators-legacy`][plugin], which mostly replicates the behavior in Babel 5 _-- back when Babel shipped with a decorator-transpiling feature._

I spoke briefly with [Brian Terlson][bter] _-- editor for the ECMAScript standard --_ about what may be changing in the decorator specification in the future. He mentioned **the decorator API may change in the future**, and that there may be changes in the execution model. In other words, what may change boils down to: the parameters you receive and order of operations.

The specification is, however, unlikely to change fundamentally. The essence of ECMAScript decorators will, in all likelihood, not change from what we've discussed in the article.

[prop]: https://github.com/wycats/javascript-decorators/blob/caf8f28b665333dc39293d5319fe01f01e3e3c0f/README.md "wycats/javascript-decorators on GitHub"
[defpp]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperties "Object.defineProperties documentation on MDN"
[ruo]: https://github.com/runuo/runuo "runuo/runuo on GitHub"
[commandpropdef]: https://github.com/runuo/runuo/blob/3f5678f061aa1b6e4d8653ae66c667d1c673e96f/Server/Attributes.cs#L179 "RunUO Attributes on GitHub"
[commandprops]: https://github.com/runuo/runuo/search?utf8=%E2%9C%93&q=CommandProperty "CommandProperty Targets on GitHub"
[uo]: http://uo.com/screenshots/ "Screenshots of Ultima Online"
[propgump]: https://i.imgur.com/NydsbSG.png
[pm]: https://github.com/runuo/runuo/blob/3f5678f061aa1b6e4d8653ae66c667d1c673e96f/Scripts/Mobiles/PlayerMobile.cs#L268-L279 "PlayerMobile.cs on GitHub"
[gca]: https://msdn.microsoft.com/en-us/library/kff8s254(v=vs.110).aspx "MemberInfo#GetCustomAttributes method on MSDN"
[gcaruo]: https://github.com/runuo/runuo/blob/3f5678f061aa1b6e4d8653ae66c667d1c673e96f/Scripts/Commands/Properties.cs#L73 "Getting an object's properties in RunUO"
[symbols]: /articles/es6-symbols-in-depth "ES6 Symbols in Depth on Pony Foo"
[maps]: /articles/es6-maps-in-depth "ES6 Maps in Depth on Pony Foo"
[protocols]: /articles/es6-symbols-in-depth#defining-protocols "Defining Protocols Using ES6 Symbols on Pony Foo"
[duelp]: http://www.runuo.com/community/threads/kenkos-duel-pit-system-working-translated.84993/ "Kenko's Duel Pit System"
[spread]: /articles/es6-spread-and-butter-in-depth "ES6 Spread and Butter in Depth on Pony Foo"
[plugin]: https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy "loganfsmyth/babel-plugin-transform-decorators-legacy on GitHub"
[babelstate]: http://phabricator.babeljs.io/T2645 "Implement new decorator proposal when finalized"
[bter]: https://twitter.com/bterlson "@bterlson on Twitter"
