# A Browserify Walkthrough

Building modules for the front-end has become an increasingly easy problem to solve. Back in the nineties we had our Java applets, our `<MARQUEE>` and `<BLINK>` tag combinations, and those beloved `<CENTER>` tags. Oh and we were mostly developing on Front Page. Anyways, time to **wean off the nostalgia**. Let's focus.

This time around I want to focus on [Browserify][2], a lean build step you can take to get [CommonJS][3] modules in your browser today. You have no idea what CommonJS modules are or why you need them? _Keep on reading!_

[![browserify.png][1]][2]

CommonJS is a module format for JavaScript, widely adopted by the [Node.js][5] community and popularized by [npm][4], the package manager that's bundled with Node. In this article you'll learn about CommonJS and how you can bring those same modules to your client-side code, using [Browserify][2] and maybe a build tool to automate your processes.

  [1]: http://i.imgur.com/tYvbZwb.png
  [2]: http://browserify.org/ "Browserify lets you require('modules') in the browser by bundling up all of your dependencies"
  [3]: http://wiki.commonjs.org/wiki/CommonJS "CommonJS Module Spec"
  [4]: https://www.npmjs.org/ "Node Packaged Modules"
  [5]: http://nodejs.org/ "Node.js Platform"
