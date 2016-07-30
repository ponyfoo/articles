The `async` / `await` feature didn't make the cut for ES2016, but that doesn't mean it won't be coming to JavaScript. At the time of this writing, it's [a *Stage 3* proposal][1], and actively being worked on. The feature is [already in Edge][3], and *should it land in another browser* [it'll reach Stage 4][2] _-- paving its way for inclusion in the next Edition of the language [(see also: TC39 Process)][4]._

We've heard about this feature for a while, but let's drill down into it and see how it works. To be able to grasp the contents of this article, you'll need a solid understanding of promises and generators. These resources should help you out.

- [ES6 Overview in 350 Bullet Points](/articles/es6)
- [ES6 Promises in Depth](/articles/es6-promises-in-depth)
- [ES6 Generators in Depth](/articles/es6-generators-in-depth)
- [Asynchronous I/O with Generators & Promises](/articles/asynchronous-i-o-with-generators-and-promises)
- [Promisees Visualization Tool](http://bevacqua.github.io/promisees/)

[1]: https://github.com/tc39/ecma262/tree/82bebe057c9fca355cfbfeb36be8e42f18c61e94 "tc39/ecma262 on GitHub"
[2]: https://twitter.com/bterlson/status/692464374842290176 "@bterlson on Twitter"
[3]: https://blogs.windows.com/msedgedev/2015/09/30/asynchronous-code-gets-easier-with-es2016-async-function-support-in-chakra-and-microsoft-edge/ "Asynchronous code gets easier with ES2016 Async Function support in Chakra and Microsoft Edge"
[4]: https://tc39.github.io/process-document/ "The TC39 Process Document"
