> Like I did in previous articles on the series, I would love to point out that you should probably [set up Babel][1] and follow along the examples with either a REPL or the `babel-node` CLI and a file. That'll make it so much easier for you to **internalize the concepts** discussed in the series. If you aren't the _"install things on my computer"_ kind of human, you might prefer to hop on [CodePen][2] and then click on the gear icon for JavaScript -- _they have a Babel preprocessor which makes trying out ES6 a breeze._ Another alternative that's also quite useful is to use Babel's [online REPL][3] _-- it'll show you compiled ES5 code to the right of your ES6 code for quick comparison._

Before getting into it, let me [_shamelessly ask for your support_][4] if you're enjoying my [ES6 in Depth][5] series. Your contributions will go towards helping me keep up with the schedule, server bills, keeping me fed, and maintaining **Pony Foo** as a veritable source of JavaScript goodies.

Thanks for reading that, and let's go into _Promises_ in ES6\. Before reading this article you might want to read about [arrow functions][6], as they're heavily used throughout the article; and [generators][7], as they're somewhat related to the concepts discussed here.

I also wanted to mention _Promisees_ -- a [promise visualization playground][8] I made last week. It offers in-browser visualizations of how promises unfold. You can run those visualizations step by step, displaying how promises in any piece of code work. You can also record a gif of those visualizations, a few of which I'll be displaying here in the article. Hope it helps!

[![An animation showing a complicated promises visualization][9]][8]

If that animation looks insanely complicated to you, read on!

[1]: /articles/universal-react-babel#setting-up-babel
[2]: http://codepen.io/
[3]: http://babeljs.io/repl/
[4]: https://www.patreon.com/bevacqua
[5]: /articles/tagged/es6-in-depth
[6]: /articles/es6-arrow-functions-in-depth
[7]: /articles/es6-generators-in-depth
[8]: http://buff.ly/1NyjC7k
[9]: https://i.imgur.com/XiRxWop.gif
