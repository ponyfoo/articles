This time around I want to focus on [Browserify][1], a lean build step you can take to obtain [CommonJS][2] modules in your browser **today**. You have no idea what CommonJS modules are or why you need them? _Keep on reading!_

[![browserify.png][3]][1]

CommonJS is a module format for JavaScript, widely adopted by the [Node.js][4] community and popularized by [npm][5], the package manager that's bundled with Node. In this article you'll learn about CommonJS and how you can bring those same modules to your client-side code, using [Browserify][1] and maybe a build tool to automate your processes.

Browserify is one of those tools that the open-source community almost takes for granted, whereas it's overlooked by the vast majority of companies, hardly making its way into projects in a professional setting. Not all is lost, sometimes big companies _do the right thing_ and use it to develop [their open-source projects][6].

If you're still stuck with [AMD modules][7] and their sheet-piles of documentation at work, then maybe you should consider Browserify for a change.

[1]: http://browserify.org/
[2]: http://wiki.commonjs.org/wiki/CommonJS
[3]: https://i.imgur.com/tYvbZwb.png
[4]: http://nodejs.org/
[5]: https://www.npmjs.org/
[6]: http://requirejs.org/
[7]: https://github.com/facebook/react/
