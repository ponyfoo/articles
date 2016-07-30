Before **JavaScript-to-JavaScript** transpilers became a _(serious)_ thing, there were modules that would add specific bits of ES6 functionality to your apps. There were things like [`gnode`][1], which allows you to use generators in `node` by interpreting your code during runtime _(or turning on the Harmony flag for generators in `node >= 0.11.x`)_

> _Trivia question: how many different names has ES6 accrued over the years?_

Then we also started to see libraries that implemented ES6 module loading, such as [`es6-module-loader`][2]. These libraries helped advance the spec by giving developers something to chew on as implementations started cropping up. Of course, you could always use CoffeeScript or TypeScript back then which had implemented language features equivalent to those in ES6.

I didn't care for the syntax in CoffeeScript, nor the fact that it would've effectively reduced my ability to [contribute to open-source][3], so that one was out. TypeScript would've been okay but they have many features on top of what's coming with ES6, and _where possible_ I try to learn things that will be **useful to me for a long time**. That being said, both of these languages contributed to the shaping of ES6, so we have them to thank for that. There's also the fact that for a long time, they were as close as you could get to trying a language with anything resembling the features in ES6.

Eventually, transpilers made an appearance. The first one was [Traceur][4], and it came out around a time where the spec wasn't locked down yet. It was constantly changing so it wasn't a very good idea to try and use it for more than a few minutes to toy around with the syntax. I got frustrated very quickly while writing [example code][5] for my [application design book][6]. Around the same time, [6to5][7] started making waves and there was also [`esnext`][8], but `esnext` never implemented ES6 modules. Earlier this year [those projects merged][9] into what we know as [Babel][10] today.

> _Come june, [the spec was finalized][11]._

[![ECMAScript 2015 Spec][12]][11]

Locking down the language features was crucial for adoption. It meant that compilers could now finally implement something and _not have stale syntax_ within the next month. The spec being finalized and Babel becoming the de-facto _JavaScript-to-JavaScript_ build tool got me interested in ES6 once again, so I started experimenting with them again.

We now have the ability to mix Browserify and Babel using `babelify`. We can use `babel-node` on the server during development -- _and compile to ES5 for production because performance reasons_. We can use Webpack if we're into [CSS Modules][13], and there's a bunch of ES6 features ready for us to use. _We need to be careful not to overplay our hand, though._ With this much going on, it's going to be hard trying to keep up while maintaining a high quality codebase that doesn't get every single new feature and shiny toy crammed into it just because we can.

> There's plenty of room in front-end tooling for feature creep, unfortunately, but **we need to battle against that** now.
>
> Tomorrow I'll be publishing an article about the parts of the future of JavaScript I'm most excited about and the concerns I have about mindlessly adopting ES6 features.

  [1]: https://github.com/TooTallNate/gnode "TooTallNate/gnode on GitHub"
  [2]: https://github.com/ModuleLoader/es6-module-loader "ModuleLoader/es6-module-loader on GitHub"
  [3]: http://bevacqua.io/opensource "I contribute on many open-source modules"
  [4]: https://github.com/google/traceur-compiler "google/traceur-compiler on GitHub"
  [5]: https://github.com/buildfirst/buildfirst/tree/master/ch05/17_harmony-traceur "'Harmony through Traceur, using Grunt' code sample for JavaScript Application Design"
  [6]: http://bevacquia.io/buildfirst "JavaScript Application Design"
  [7]: https://www.npmjs.com/package/6to5 "6to5 on npm"
  [8]: https://github.com/esnext/esnext "esnext/esnext on GitHub"
  [9]: http://babeljs.io/blog/2015/01/12/6to5-esnext/ "6to5 + esnext"
  [10]: http://babeljs.io/ "Babel JavaScript Compiler"
  [11]: http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf "ECMA-262 6th Edition was standarized on June 2015"
  [12]: https://i.imgur.com/SiBLQvN.png "Screen Shot 2015-08-25 at 17.37.57.png"
  [13]: http://glenmaddern.com/articles/css-modules "CSS Modules article by Glen Maddern"
