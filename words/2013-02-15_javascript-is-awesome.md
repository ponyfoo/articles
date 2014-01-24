# JavaScript Is Awesome #

_JavaScript_ is one of the most **loved and hated** languages out there. Some, can't stand the stench of how obtuse it _appears_ to be. Some appreciate the "anything goes" virtues of an interpreted language, and would marry _JavaScript_ given the chance.

Behold: the **"hate speech"**.

- Interpreted languages are too bug-prone
- **Callback hell**, or how _easily_ you can lose track of what's going on
- Awkward **equality comparers**, `==`, `===` are infuriating
- `this` is a mess
- **Anything goes**. Loose typing, how pretty much everything can mutate into anything without notice
- Nonsensical behavior like `parseInt('08') === 0` [(this one changed recently for V8)](http://code.google.com/p/v8/issues/detail?id=1645 "V8 Issues - parseInt still parsing octal")
- Poor support for (classical) **debugging**, such as _reasonable_ stack traces
- **Prototypal** vs _Classical_ inheritance
- [All things JavaScript suck](http://java.dzone.com/articles/f-mongodb-f-nodejs-and-f-you "F MongoDB, F Node.js, and F You!")

> I **disagree** with most of the above, and I want to spend _a few words_ addressing these concerns.

First of all, the _vast majority_ of these concerns arise from the paradigm shift people experience when developing in _JavaScript_. Most people are accustomed to a compiler telling them when something is amiss. In _JavaScript_, however, there is no compiler, _JavaScript_ is interpreted as it is executed. There are a few ways to work around (or _mitigate_) this concern.

One way to _mitigate_ the impact of not being able to compile your code, is [`use strict`](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Functions_and_function_scope/Strict_mode "Strict mode explained"). By the way, [MDN](https://developer.mozilla.org/en-US/docs/JavaScript "Mozilla Developer Network") is an **extensive resource library** on _JavaScript_ you are _not using enough_.

`use strict` in _JavaScript_ is reminiscent of [Option Explicit](http://msdn.microsoft.com/en-us/library/y9341s4f(v=vs.80).aspx "MSDN - Option Explicit"), back in the _Visual Basic 6_ days. It's merely a safeguard against silly mistakes, but it comes in handy.

A more **definitive** approach to solve the issue is to resort to a _JavaScript_ compiler, such as [CoffeeScript](http://coffeescript.org/ "CoffeeScript Language"), or [TypeScript](http://www.typescriptlang.org/ "TypeScript Language").

> I discourage taking this approach. Not only because languages like _CoffeeScript_ tend to be **very opinionated** about how your code should look like, but most importantly, it isn't _JavaScript_, it's an _entirely different_ language; even though it **incidentally** compiles to _JavaScript_.

> The notion of using _TypeScript_ is slightly less infuriating, since it's a _superset_ of _JavaScript_, rather than a different language.

Having said that, **the fact that both compilers output pure _JavaScript_ remains**, which is reason enough to learn how to code in _JavaScript_ properly by yourself, rather than have a layer of abstraction making coding style choices for you. It's not like _JavaScript_ is [MSIL](http://en.wikipedia.org/wiki/Common_Intermediate_Language "Microsoft Intermediate Language"). _MSIL_ is **really hard** to read, maintain, write, and everything in between, and generally _not worthwhile to code in_, unless you need to _fine-tune performance_, dynamically _generate classes_, or the like.

A ["simple" program](http://www.dotnetperls.com/il "Example Source") in _MSIL_ might be:

```
.method private hidebysig static void Main() cil managed
{
	.entrypoint
	.maxstack 2
	.locals init (
	[0] int32 num)
	L_0000: ldc.i4.0
	L_0001: stloc.0
	L_0002: br.s L_000e
	L_0004: ldloc.0
	L_0005: call void [mscorlib]System.Console::WriteLine(int32)
	L_000a: ldloc.0
	L_000b: ldc.i4.1
	L_000c: add
	L_000d: stloc.0
	L_000e: ldloc.0
	L_000f: ldc.i4.s 10
	L_0011: blt.s L_0004
	L_0013: ret
}
```

 _JavaScript_ is not _that_ **obscure**. _JavaScript_ can be learnt. Compilers _hinder_ your ability to do that. You should learn what a _closure_ is. How to extend a _prototype_. How to use _callbacks_ sensibly. There lies the real beauty of _JavaScript_ code.

# Demystifying _JavaScript_ #

> - Interpreted languages are too bug-prone

While it might be true that _JavaScript_ code is harder to debug, and can definitely be _harder to maintain when improperly structured_. There are several tools at our disposal to prevent chaos.

You should treat _JavaScript_ as you would any other language, and as such, **you should Unit Test your code**. There are [several](http://pivotal.github.com/jasmine/ "Jasmine BDD Test Framework") [frameworks](http://visionmedia.github.com/mocha/ "Mocha Test Framework") [to](http://vowsjs.org/ "Bows BDD Test Framework") [pick](http://qunitjs.com/ "QUnit by jQuery") [from](http://developer.yahoo.com/yui/yuitest/ "YUI Test from Yahoo").

Client-side _JavaScript_ might prove harder to structure than _JavaScript_ in Node.JS. Frameworks like [Backbone.js](http://backbonejs.org/ "Backbone MVC Framework") and [Knockout.js](http://knockoutjs.com/ "Knockout MVVM Framework") can help you structure your code cleanly and without making an unmaintainable mess out of it.

> - Awkward **equality comparers**, `==` is infuriating

This one I'll concede. The _equality operators_, `==` and `!=` are a mess.. They try too hard. If the values being compared are of _different types_, these operators will attempt to **coerce** these values using complicated rules that are _not even transitive_. Avoid them entirely. Use, [as Crockford puts it](http://www.amazon.com/dp/0596517742 "JavaScript: The Good Parts"), the "good operators": `===` and `!==`. These return false in case of type mismatch, as would be expected.

> - `this` is a mess

We know how `this` story goes. I'll illustrate with a bad example:

```js
function Car(opts){
	this.wheels = opts.axles * 2;
	return this;
}

var compact = new Car({axles: 2});
var truck = Car({axles: 5 });
```

Now _compact_ is a four-wheel _car_, while our _Window_ became a _truck_, and suddenly has 10 _wheels_. Furthermore, `this` can be chosen through `Function.prototype.apply`, and otherwise changes depending on the **context** it's being _evaluated_ in.

Well, you have two options. One option is to _ignore the damn thing_, and don't use `this` or `new`, after all, **ignorance is a bliss**. The other option is, 
_you guessed it_, [learning how it works](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Operators/this "this operator").

> - **Anything goes**. Loose typing, how pretty much everything can mutate into anything without notice

This one is, in my opinion, one of the **biggest selling points** in favor of _JavaScript_, which is more often confused with a _drawback_ against the language. I'd rephrase this as **_JavaScript_ just works**, in that whatever change you want to introduce to your code, _JavaScript_ can accomodate. And as long as you are _conscious_ about maintainability, such flexibility shouldn't become a problem.

> - Nonsensical behavior like `parseInt('08') === 0` [(this one changed recently for V8)](http://code.google.com/p/v8/issues/detail?id=1645 "V8 Issues - parseInt still parsing octal")

There are many oddities like this in _JavaScript_, more than in some other languages such as _C#_, fewer than in [others](http://php.net/ "PHP"). Most of these oddities are _easily avoided_, and their persistance stems from **poor implementations** in some web browsers. Others (such as `for..in` problems), simply come from the nature of a prototypal inheritance model.

> - Poor support for (classical) **debugging**, such as _reasonable_ stack traces

Often overlooked is the ability to **execute code on demand** in a browser _console_, or writing debug messages which you can see in real time with _no extra frameworks_, and without altering the user experience. These are things you often can't do in other languages either, once you get accustomed to these, you'll find it _way more productive_ to debug with a console and hacking away at the DOM, than to step through your code, and recompiling every time you add a comma.

> - **Prototypal** vs _Classical_ inheritance

In _JavaScript_, _things_ inherit from other _things_, and not from abstract concepts _(or classes)_. This might be hard to wrap your head around if you are experienced in class-based languages such as _Java_ or _C#_, but you'll soon _get over it_, and begin [reaping the benefits](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Inheritance_and_the_prototype_chain "Inheritance and the prototype chain").

### Conclusion ###

_JavaScript_ is awesome, and you should **love it** more!