# Managing Code Quality in NodeJS #

I've mentioned [CI](/2013/01/18/continuous-integration-and-automated-deployments "Continuous Integration, and Automated Deployments") and [static asset management](/2013/01/18/asset-management-in-node "Asset management in Node") in the past. Now I want to talk about code quality.

This article is mostly a follow up on the [CI](/2013/01/18/continuous-integration-and-automated-deployments "Continuous Integration, and Automated Deployments") post. I'll describe how [Grunt](gruntjs.com "Grunt: The JavaScript Task Runner") helped me change the _test_ and _build_ processes used in this blog's [engine](https://github.com/bevacqua/ponyfoo "ponyfoo blogging engine").

> Before using Grunt, I didn't really have a **real** build process. _Sure_, `git push heroku master` _triggered a build_ on their end, but I didn't control any of it, all I did was `node app.js`.

Similarly, my [Travis-CI hook](https://travis-ci.org/bevacqua/ponyfoo/builds "Travis Build Log") just made sure there weren't any conflicts with my npm packages. I could do _better_.

I've started leaning towards making the engine more _testable_, I figured I had to start **somewhere**. For me, that first step was to _use a task runner_.

# [Grunt](gruntjs.com "Grunt: The JavaScript Task Runner"): proper Build Process #

After spending a few days toying with Grunt, I must say it's _really exciting_ to use a build tool that _just works_, rather than get in your way. I learned a couple of interesting things, and even created my very own [grunt plugin for assetify](https://github.com/bevacqua/grunt-assetify "grunt-assetify on GitHub").

Grunt runs from the command line, and you should install it globally, using `npm install -g grunt-cli`. Grunt will look for `Gruntfile.js`, and run the function exported by that module, passing a helper as an argument.

Here's a sample listing that does basically the same as the traditional `node app.js` command used to. Save this as `Gruntfile.js`:

```js
'use strict';

module.exports = function(grunt) {
    grunt.initConfig({});
    grunt.registerTask('app', function(){
        var done = this.async(),
            app = require('./app.js');

        app.start(done);
    });
};
```

You should make a very minor change to your `app.js`, and add `app.on('close', done);`. This will signal grunt to shut down when the server is closed, rather than as soon as it's idle.

Now you can `grunt app` to run your web application using `grunt`. How cool is that? _Fine_, it's not very amusing.

The amusing part is the kind of things Grunt _enables_ you to do, now that it's set up. You can _integrate_ other stuff you might've shied away from because it was too cumbersome to run, or complicated to configure. Grunt solves some of that for you.

![grunt-cli-sample.png][1]
  
I'll now go over some of the tools I integrated into my _build process_ since I switched to Grunt. I'm not saying Grunt enabled me to use these tools, it just 
_made my life easier_.

## Lint: Static Code Quality Analysis ##

[JSHint](http://www.jshint.com "JSHint") is a  In case you don't know it, [JSLint](http://jslint.com/ "JSLint by Douglas Crockford") helps catch syntax errors (and _coding style issues_) in your code. JSHint is highly configurable fork of JSLint, and their [documentation](http://www.jshint.com/docs/ "JSHint Documentation") explains each option in detail.

You should consider JSHint as your very first unit test. Since JavaScript doesn't have a formal compiler, a linting tool such as JSHint will have to do.

You can find the GitHub repository to their Grunt plugin [here](https://github.com/gruntjs/grunt-contrib-jshint "JSHint plugin for Grunt"), and the actual sources of JSHint are [here](https://github.com/jshint/jshint "JSHint on GitHub").

Here's how I added it to my `Gruntfile.js` (displaying only the relevant code):

```js
grunt.initConfig({
    jshint: {
        node: {
            files: {
                src: [
                    'gruntfile.js',
                    'src/**/*.js',
                    '!src/static/**/*.js',
                    'test/spec/**/*.js'
                ]
            },
            options: {
                jshintrc: '.jshintrc'
            }
        },
        browser: {
            files: {
                src: [
                    'src/static/**/*.js',
                    '!src/static/config/*.js',
                    '!src/static/.bin/**/*.js',
                    '!src/static/js/vendor/**/*.js'
                ]
            },
            options: {
                jshintrc: '.jshintrc-browser'
            }
        }
    }
});

grunt.loadNpmTasks('grunt-contrib-jshint');
```
    
This brings us to an interesting point about Grunt, which is _targets_. The `node` and `browser` properties are just _targets_, which means that `grunt jshint:node` will run JSHint with the _first set_ of options, and `grunt jshint:browser` will run the _second set_. `grunt jshint` will do _both_ in turn.

If you're not _already_ linting your code, it will take you a few moments to make it comply with a default JSHint configuration, but it will be _worth it_.

## Unit Testing ##

If you're looking to get started with Unit Testing in NodeJS I'll just recommend you to read [TDD.JS](http://tddjs.com/ "Test-Driven Development in JavaScript").

I'll expand on this topic on a later post, since there's a lot to write on the topic, particularly on testing a dynamically typed language such as JavaScript, which in my opinion is both harder but more **enjoyable** (when done right).

Meanwhile, you _must_ look at [Superhero.js](http://superherojs.com/), which offers a bunch of JavaScript articles, videos, and books you should definitely keep an eye out for!

## Again with Static Asset Management ##

I've mentioned in the past how I was building [assetify](/2013/01/18/asset-management-in-node "Asset management in Node"). It hasn't changed much lately, except for a fingerprinting (as in `/all.js?v=2r4c-19by10z`) middleware that allows us to set far-future `Expires` headers. Other than that, I did add a [grunt plugin](https://github.com/bevacqua/grunt-assetify "grunt plugin for assetify") for assetify.

If you're interested in plugging assetify into your Grunt configuration, you should definitely check out the [example repository](https://github.com/bevacqua/grunt-assetify-example "grunt-assetify example usage") I've set up on GitHub. The Grunt setup on that repository also has JSHint and Jasmine tasks, so it might even be a good starting point for configuring a nice `Gruntfile.js`

Lastly, _remember_:

> Stand watchful, and continuously assess the quality of your code. These guidelines may make maintaining, refactoring, and developing your application leaner, more straightforward, and _less bug-prone_. You are the one who's going to have to stand by them. Incorporate them as a part of your everyday work, and you'll soon start to _reap the benefits_.

![continuous.gif][2]

  [1]: http://i.imgur.com/i28vdBO.png "typical grunt console output"
  [2]: http://i.imgur.com/Pzfnf7z.gif "Stand watchful"