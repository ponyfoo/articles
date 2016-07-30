The most obvious one is -- recursively -- the fact that `Promise` is now in the language. `Promise` being in the language drives adoption by itself, through the official endorsement of the spec-writing JavaScript gods. Furthermore, libraries like Babel and `bluebird` making it all too simple to include spec-compliant `Promise`-based solutions in any app at very little cost. Anyone writing code for modern browsers or using a modern development toolchain leverages Babel and/or `bluebird`.

Another reason is that **developers are increasingly comfortable with ES6**. There have been plenty of [tutorials][es6], a couple of books, and many conference talks describing ES6. It's been roughly a year since the specification was finalized. People now roughly understand the `Promise` API, and what's better: the API isn't changing anymore. Recently, [jQuery 3 was released][jq3] into *Promises/A+* compliance, a huge win for native `Promise`.

Third, there's [`async`/`await`][aa]. While not the most heavily utilized JavaScript API, `async`/`await` is already a [stage 3 proposal][stag], and at this point I think we can state confidently and without hesitation that it'll someday be an official language feature.

> The synergy between `Promise`, `async`/`await`, and generators, is just too good to pass up!

[promises]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
[es6]: /articles/es6 "ES6 Overview in 350 Bullet Points on Pony Foo"
[jq3]: https://blog.jquery.com/2016/06/09/jquery-3-0-final-released/ "jQuery 3.0 Final Released!"
[aa]: /articles/understanding-javascript-async-await "Understanding JavaScript’s async await on Pony Foo"
[stag]: https://github.com/tc39/ecmascript-asyncawait "tc39/ecmascript-asyncawait on GitHub"
