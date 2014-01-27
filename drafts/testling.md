# Continuous Browser Testing with Testling

Browsers are becoming a thing. The web platform is sprawling with popularity, and you have no idea what browsers your code works on. That's discouraging, scary, **unreliable**. By now we all know about the benefits of testing. Good test coverage makes your code more reliable, maintainable, predictable, and approachable. Going through a project's unit tests helps you give an idea of the intended behavior of a certain piece of functionality that might otherwise be poorly documented (although I'd rather [discourage maintainers from relying on tests as documentation][1]).

In this article I aim to walk you through the necessary tooling. We will write unit tests using Mocha, and run them on browsers using Testling. Testling leverages Common.JS modules to allow us to run unit tests on multiple browsers on every `git push`, which is pretty awesome.

# Think Browser Support

One of the most challenging aspects of building successful web applications is making them compatible with as many browsers as you can get away with. That means, for instance, that you shouldn't use [`should.js`][2] in your client-side unit tests, as it has getters.

```js
{
  get thing () { return 1; }
}
```

Getters blow up in the older browsers. The classical [`assert`][3] module might be a better fit for us if we want to support the usual suspects, such as `IE <= 8`.

> You're lucky you know about this before hand, otherwise you might've needed to [translate your tests away from `should.js` and into `assert`][4].

Supporting older browsers also means providing [polyfills][5], small snippets of code which enable features available only to newer browsers, in older ones. It also means being careful about which browser APIs we can use. Can you use the fullscreen API? How about the file system API? Does it browser support WebGL? Using a service such as [caniuse.com][6] can definitely help you identify which features are available in the browsers you want to support.

That being said, you won't have the same requirements in every case. You might not be developing a library, with no dependencies, which must provide polyfills for old browser support. You might actually be using such a library. Then, browser support would be resolved, for the most part. However, even if you know for a fact that _cross-browser is a non-issue_, or you only need to provide support for a single browser, you still need to have tests to make sure your code works the way you'd expect it to.




  [1]: /2014/01/20/how-to-design-great-programs "How to Design Great Programs"
  [2]: https://github.com/visionmedia/should.js "visionmedia/should.js on GitHub"
  [3]: https://github.com/defunctzombie/commonjs-assert "defunctzombie/commonjs-assert on GitHub"
  [4]: https://github.com/bevacqua/contra/commit/1ca62b377595540eeb73fdd08a7c9edcb8ded377#diff-a11d42d80e96bd0a929b68d115fc7759 "'should -> assert, better browser compat' commit on bevacqua/contra"
  [5]: http://www.2ality.com/2011/12/shim-vs-polyfill.html "What is the difference between a shim and a polyfill?"
  [6]: http://caniuse.com

[unit-testing testling browser-testing]
