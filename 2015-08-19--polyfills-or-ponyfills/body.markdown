Christian Heilmann argues that, sometimes, developers probe feature support by [testing for some other feature][1] that's implemented most of the time alongside the feature we actually want to use. Under those circumstances, and considering you already have tools like [Babel][2] if you'd like to play around with ES6, I suggest you strongly avoid polyfills for any ES6 features, out of the ones that could be polyfilled.

As an alternative, you could use _ponyfills_ instead.

## Ponyfills

A _ponyfill_ is almost the same as a polyfill, but not quite. Instead of patching functionality for older browsers, a ponyfill provides that functionality as a standalone module you can use. Let's go to an example.

Here's how your typical polyfill looks like -- it was taken from [MDN][3].

```js
if (!String.prototype.trim) {
  String.prototype.trim = function () {
    return this.replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g, '');
  };
}
```

The equivalent ponyfill would be a module that exports the method below.

```js
function trim (text) {
  return text.replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g, '');
}
```

They're very similar, except a ponyfill doesn't patch missing functionality others may be relying on for _feature detection_.

## Polyfills or Ponyfills?

I think both have their merits. Polyfills are great for methods like [`String.prototype.trim`][3] because they allow you to use the methods on `String` instances. That gets even better with methods like `.map`, and `.reduce` which return `this`, so chaining is great. Sure, you could always do `reduce(map([1, 2, 3], twice), sum, 0)` instead of `[1, 2, 3].map(twice).reduce(sum, 0)`, but there's also the fact that many of your dependencies probably are assuming ES5 is on the table, so it's useful to polyfill for the stuff that's missing in older browsers.

When it comes to ES6 (or more complicated ES5 polyfills) however, the situation changes a little bit. Sometimes we can't implement a solution that's **fully spec-compliant**, and in those cases using a polyfill might be the wrong answer. A polyfill would translate into telling the rest of the codebase that it's okay to use the feature, that it'll work just like in modern browsers, but it might not in edge cases.

In that situation it's better to use a ponyfill, because that way you won't be polluting expectations about what. Only you will be leveraging the new functionality, and you are well aware of the limitations of that solution, so _all is good_ with the world. Meanwhile, other pieces of code that are _feature-detecting_ on the piece of functionality you've half-patched will continue to work by ignoring it as usual. Here, we can fall back to a great thing I've heard that I don't hear often enough: "users browsing the web with shitty browsers <mark>are used to shitty experiences</mark>".

> In so many words, strive **not to break expectations**.

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][4]._ Remember to **subscribe** if you got this far!

[1]: http://christianheilmann.com/2015/08/17/how-about-we-make-es6-the-new-baseline/ "'How About We Make ES6 the New Baseline?' asks @codepo8"
[2]: https://github.com/babel/babel "babel/babel on GitHub"
[3]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/String/trim "String.prototype.trim() – MDN"
[4]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
