> Like I did in previous articles on the series, I would love to point out that you should probably [set up Babel][1] and follow along the examples with either a REPL or the `babel-node` CLI and a file. That'll make it so much easier for you to **internalize the concepts** discussed in the series. If you aren't the _"install things on my computer"_ kind of human, you might prefer to hop on [CodePen][2] and then click on the gear icon for JavaScript -- _they have a Babel preprocessor which makes trying out ES6 a breeze._ Another alternative that's also quite useful is to use Babel's [online REPL][3] _-- it'll show you compiled ES5 code to the right of your ES6 code for quick comparison._
> 
> Note that `Proxy` is harder to play around with as Babel doesn't support it unless the underlying browser has support for it. You can check out the [ES6 compatibility table][4] for supporting browsers. At the time of this writing, you can use _Microsoft Edge_ or _Mozilla Firefox_ to try out `Proxy`. Personally, I'll be verifying my examples using _Firefox_.

Before getting into it, let me [_shamelessly ask for your support_][5] if you're enjoying my [ES6 in Depth][6] series. Your contributions will go towards helping me keep up with the schedule, server bills, keeping me fed, and maintaining **Pony Foo** as a veritable source of JavaScript goodies.

Thanks for reading that, and let's go into more `Proxy` _traps_ now! If you haven't yet, I encourage you to read [yesterday's article on the `Proxy` built-in][7] for an introduction to the subject.

[1]: /articles/universal-react-babel#setting-up-babel
[2]: http://codepen.io/
[3]: http://babeljs.io/repl/
[4]: http://kangax.github.io/compat-table/es6/
[5]: https://www.patreon.com/bevacqua
[6]: /articles/tagged/es6-in-depth
[7]: /articles/es6-proxies-in-depth
