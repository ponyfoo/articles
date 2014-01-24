# Is WebDriver as good as it gets?

This blog post is part _rant_, part _learning experience_, and part _solutions and conclusions_ I've arrived at, while struggling with [WebDriver][5] implementations in Node.

[Integration Testing][1] is a must if we're to build a reasonably resilient web application. It helps us figure out evident errors in the "happy path" of execution. However, when it comes to automated tools that enable us to do integration testing **on real browsers**, _the options available to us are pretty underwhelming_.

WebDriver was [introduced by Google in 2009][2].

> WebDriver is a clean, fast framework for automated testing of webapps.

To my knowledge, _documentation for WebDriver is scarce_, at best. Attempts to interact with it are going to be painful for you, particularly if you're attempting to use one of the even less documented implementations, such as the [wd][3] package, built to run these tests using Node. The worst of it is that there **doesn't seem to be a better alternative** if you want to test with real browsers, like Chrome _(which you should)_. This is paired with the popularity of partially-implemented libraries which _"sort of do what you want"_, but aren't able to do really basic stuff like handling file uploads.

I wish I had the time to invest effort in a Kickstarter project to improve the current state of affairs. I'd love to see a better integration testing solution which can be implemented in any language through an API like Selenium does, and supports any browser, **which just works**, and whose consuming libraries were a lot better (API wise), and much better documented as well. Good documentation is _vastly underestimated_ nowadays.

[![selenium.png][4]][5]

  [1]: http://en.wikipedia.org/wiki/Integration_testing "Integration Testing on Wikipedia"
  [2]: http://google-opensource.blogspot.com.ar/2009/05/introducing-webdriver.html "Introducing WebDriver"
  [3]: https://github.com/admc/wd "admc/wd on GitHub"
  [4]: http://i.imgur.com/T5uFEMC.png
  [5]: http://docs.seleniumhq.org/projects/webdriver/ "Selenium WebDriver"

### A Safety Net

I was to automate a testing process we were doing, where we basically had a checklist of items that needed to be validated, before we could sign off on a deployment for production. The list looked sort of like this:

1. Log in
2. Create a project and edit it
3. Upload a file
4. Create a view using the uploaded file
5. Validate a thumbnail is generated
6. Log into mobile application
7. Preview the new project
8. Delete the project
9. Log out

This kind of testing helps us make sure we don't deploy silly mistakes to production. At a bare minimum, which is what this checklist represents, **frequently used features should just work**. Testing, in all its flavors, amounts to nothing if it's not automated. After manually going through the checklist for a couple of weeks, it was clear we needed to start automating it.

[Unit Tests][4] are, of course, necessary to catch more subtle issues. Integration however aims for gross oversights, and generally being able to actually execute the application.

I gave [wd][1] a shot, paired with [grunt-mocha-webdriver][2] so that we could automate it using [Grunt][3].

### Grunt Automation

First off, we need to install the grunt task.

```shell
npm install --save-dev grunt-mocha-webdriver
```

Then, setting up the Grunt task was relatively easy.

```js
mochaWebdriver: {
  options: {
    timeout: 1000 * 60,
    reporter: 'spec'
  },
  integration: {
    src: ['test/integration/**/*.js'],
    options: {
      usePhantom: true,
      usePromises: true
    }
  }
}
```

A couple of things to mention, in retrospect. At first we thought using Phantom, instead of a real browser, would be _such a good idea_, because it'd be faster to set up, and what not. A co-worker convinced me to use promises, and I went with it. The task could definitely use a better name, but at least it worked. So I had something to get me going.

There was one issue, though. I had to fire up the Node application myself, which meant extra work every time. I don't like extra work, so I wrote a task that would start a Node process, run the tests, and then stop that process. I decided that I needed to wait on the application to start listening for requests, rather than shoving them to its face. I used [a little port watching module][5] which notifies me when an application starts listening on a given port, which worked just fine.

```js
var app;

grunt.registerTask('integration-test-runner:start', function () {
  var done = this.async();
  var _ = require('lodash');
  var spawn = require('child_process').spawn;
  var finder = require('process-finder');
  var port = process.env.TEST_PORT || 3333;
  var watcher = finder.watch({ port: port, frequency: 400 });
  var env = _.clone(process.env);

  console.log('Spawning node process to listen on port %s...', port);

  env.PORT = port;
  app = spawn('node', ['app'], { stdio: 'inherit', env: env });

  process.on('exit', close);

  watcher.on('listen', function(pid) {
    console.log('Process %s listening on port %s...\nRunning tests.', pid, port)

    grunt.task.run(
      'mochaWebdriver:integration',
      'integration-test-runner:cleanup'
    );
    done();
  });
});

grunt.registerTask('integration-test-runner:cleanup', close);

function close () {
  if (app) {
    app.kill('SIGHUP');
    console.log('Process %s shutting down', app.pid);
  } else {
    console.log('Process not found');
  }
}
```

Note that `grunt.task.run` won't actually run the task in place, but _enqueue_ it so that it runs _after_ the currently executing task, which is why `done` is called immediately afterwards. Giving the app an uncommon port was useful because I could run the application in port `3000`, like it does by default, side-by-side with the automated tests. Lastly, the cleanup logic helped me avoid issues with the Node process taking over the port, preventing those infamous `EADDRINUSE` errors.

### The First Test

Writing the first test was kind of awkward because I wasn't really familiar with promises [(did you know they're coming to JavaScript in Harmony?)][6], but I decided to give them a shot, anyway. Here is the first test, trying out the login logic.

```js
var port = process.env.TEST_PORT || 3333;
var base = 'http://localhost:' + port;

it("handles an invalid login", function (done) {
  this.browser
    .get(base + '/admin')

    // assert we're taken to the login page
    .url().then(function (url) {
      console.log('Logging in...');
      return assert.equal(url, base + '/login');
    })

    // enter invalid credentials and submit
    .elementById('email').type('me@here.com')
    .elementById('pass').type('wayoff')
    .elementByTagName('form').submit();

    // assert we're still in the login page
    .url().then(function (url) {
      return assert.equal(url, base + '/login');
    })

    .then(done, done);
});
```

Looks really simple, innocent, and straightforward, right? _I know!_ Thing is that actually getting it to work took a lot of effort, because the API is so underdocumented, I actually had to log `Object.keys(this.browser)` and go through the methods, trying to figure out which one did what I intended to do (submit the form, or type into an input). These are all symptoms of a lousy documentation. The API _could be worse_, as it at least attempts to mirror some of the methods [found in the native DOM][7].

### Context Barrier

Suppose you get past this initial barrier. Then there's the context issues, and understanding how to transverse the DOM properly. If you want to get an element at random, navigate away, and then come back and select the same element, you're gonna have a bad time. 

This one, however, is mostly a matter of befriending promises and accumulating experience with the `wd` API. In particular, the way `.then` statements work is pretty cryptic. If they return an element, then that'll be the context for the next promise in the chain, if they don't then the `browser` object is used, and if you prepend `'>'` to selector requests, the context is used to restrict the query. For example:

```js
this.browser
  .elementByCssSelector('.some-thing')
  .elementByCssSelector('>', '.some-thing-child')
  .then(function (element) {
    // return the element or you're back to the browser
    return element;
  });
```

Obviously, though, when you go for something like `.text().then(function (value) {})`, you're pretty much screwed because you don't have a reference to the element anymore, unless you've previously persisted it to a variable.

### Capturing Page Load Errors

You'd think capturing page load errors is something that anyone would like to use. That is, logging errors that happen right after a request completes, during interpretation or execution of a piece of code. Well, if you google around, the only real solution to this problem is using client-side JavaScript, and patching [onerror][13] so that you can keep track of errors.

```js
window.__wd_errors = [];
window.onerror = function (message, url, ln) {
  window.__wd_errors.push({
    message: message,
    url: url,
    ln: ln
  });
};
```

As WebDriver doesn't really provide a mechanism to inject into the response stream, or manipulate it in any way _(that I could find, anyways)_, you're stuck with patching the application if a `TEST_INTEGRATION` environment variable or similar is turned on. On the testing side of things, you could augment the promise chain prototype to print the list of errors, after navigating to a page.

```js
wd.PromiseChainWebdriver.prototype.throwIfJsErrors = function () {
  return this
    .safeEval('window.__wd_errors')
    .then(function (errors) {
      if (errors && errors.length) {
        console.log('Detected %s Error(s)', errors.length);

        errors.forEach(function (error) {
          console.log('%s\nAt %s, File %s', error.message, error.ln, error.url);
        });
      }
    });
};
```

This helped me catch an unexpected issue. Phantom doesn't have `Function.prototype.bind`, and [it won't get included until `2.0.0`][14], which _doesn't seem to be happenning any time soon_. Temporarily, I added a polyfill for `Function.prototype.bind` to the file which had the the page load error capturing.

### File Uploading _is a Nightmare_

The worst offender of all, was file uploading. Of course, _documentation would've helped_. It's like nobody wants to talk about integration testing anyways, so googling around doesn't do you a lot of good either. The best I could come up with was some information on [a discussion on an issue on GitHub][8], and maybe questions about the WebDriver implementation in Java, which _I attempted to mirror_.

I tried everything. At first, I went with the API: find the element, then use `.sendKeys(<path>)`. Nope, that won't work. Okay, maybe it was just `.type(<path>)`? No. Need a `.click()` in between? Wrong again. You see, the lack of documentation makes it very hard for the consumer to know exactly how wrong their approach is. That represents a huge problem, because **you'll keep on trying things out blindly**, hoping to eventually get it right. **You won't.**

After lots of googling and a some desperate attempts, I stumbled upon a corner of the internet where I read that this functionality [wasn't implemented a few months ago][8], sure [this pull request][9] sounds like it should be working, but [this issue on GhostDriver suggests otherwise][10], and I got tired of sifting through issue lists figuring out whether what I was trying to do was even supported.

Okay, great, so I decided to try something else. I know! I'm fine not testing the button click itself, **that's something a unit test could do**. I care about the grand scheme of things. I _need to upload the file_, though. There's no getting around that. Luckily, the page I was testing wasn't submitting the form directly. It creates a [FormData][11] object, and places the files there, and then it sends that. All I need is to `eval` the right string, and it'll all be over!

Some googling gave me the formula. Rather than give the file path to WebDriver, I had to hand over the `Blob` data directly to the browser. Converting the file to the correct format took a little trial and error, but the renewed spirit was there. Here's a small addition to the `wd` promise chain prototype, so I could re-use my awesome file upload hack, some day.

```js
wd.PromiseChainWebdriver.prototype.uploadSomething = function (file, scope) {
  console.log('Attempting file upload...');

  var mime = require('mime');
  var name = path.basename(file);
  var blob = fs.readFileSync(file, { encoding: 'base64' });
  var code = [
    util.format('var bytes = atob("%s");', blob),
    'var codepoints = Array.prototype.map.call(bytes, function (n) { return n.charCodeAt(0); });',
    'var data = new Uint8Array(codepoints);",
    util.format('var blob = new Blob([data], { type: "%s" });', mime.lookup(file)),
    util.format('blob.name = '%s';', name),
    'var formData = new FormData();',
    util.format('formData.append("%s", blob);'
  ].join(' ');

  return this
    .safeEval(code)
    .then(function () {
      console.log('File upload in progress.');
    });
};
```

Feeling _great!_ Let's do this! ...nope, _not working_. [Blob is unsupported in Phantom < `2.0.0`][12], just like [`Function.prototype.bind`][15]. The [polyfill][16] I tried out didn't work either, and I simply moved on to Chrome.

### Moving to Chrome

Okay, fine. Rather than using the unreliable Ghost browser, I needed the real thing, and so I went with Chrome. To run tests with Chrome, I needed to change things up quite a bit. First off, I found a [really simple selenium server installer][17] which did the trick of firing up a Selenium server for me.

```shell
npm install -g selenium-standalone
```

Now I could fire up an instance in my command line. _By the way, it requires Java!_

```shell
start-selenium
```

Wait a minute... that's too easy! Oh yeah, that's right, `grunt-mocha-webdriver` doesn't run against local selenium instances, even though [a pull request][18], which adds that functionality, **is already a month old**. I went ahead and [created a package][19] out of the pull request, using that I could test against the local selenium instance created by `start-selenium`.

I really wanted to keep the selenium instance contained in the Grunt task, as well, so I went ahead and [cloned `selenium-standalone`][20], adding a programmatic API. After that, it was just a matter of pulling it into a new Grunt task. That task would spawn a selenium server instance consuming the API I've just written.

```js
var selenium = require('selenium-standalone-painful').start({
  stdio: 'pipe'
});

var ready = /Started org\.openqa\.jetty\.jetty\.Server/i;

selenium.stdout.on('data', function () {
  if (ready.test(data.toString())) {
    grunt.task.run('the-next-one');
  }
});

process.on('exit', function () {
  if (selenium) {
    selenium.kill('SIGHUP');
  }
});
```

That's it! Then I decided to improve the reusability by pulling it out of its host project.

### Introducing `grunt-integration`

I built a tool specifically to deal with the <del>issues</del> <ins>ordeal</ins> I went through while ramping up on my integration testing experience on Node applications. Concretely, these are the features I want in an integration testing module, and also the goals of [`grunt-integration`][21]:

- Start a local selenium server instance
- Start a local program, such as `node` application
- Wait for the program to listen on a specific port
- Execute integration tests using [Mocha][22] and [WebDriver][1]
- Using real browsers, such as Chrome
- Automatically, in one command
- Less painful installation process, please!

I'm considering adding some extensions to `wd` so that dealing with some of the issues I've described here is not so painful. The `wd` API could definitely get some love, but you can't do a lot better than what it currently has. HTTP injection would be something that I'd love to see there, but I don't think its even possible with Selenium.

  [1]: https://github.com/admc/wd "admc/wd on GitHub"
  [2]: https://github.com/jmreidy/grunt-mocha-webdriver "jmreidy/grunt-mocha-webdriver on GitHub"
  [3]: http://gruntjs.com/ "Grunt: The JavaScript Task Runner"
  [4]: /2013/03/28/pragmatic-unit-testing-in-javascript "Pragmatic Unit Testing in JavaScript"
  [5]: https://github.com/bevacqua/process-finder "bevacqua/process-finder on GitHub"
  [6]: http://www.html5rocks.com/en/tutorials/es6/promises/ "JavaScript Promises: There and back again"
  [7]: /2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API"
  [8]: https://github.com/admc/wd/issues/107 "Add file upload support"
  [9]: https://github.com/admc/wd/pull/122 "Upload local file to remote server support"
  [10]: https://github.com/detro/ghostdriver/issues/155 "Problem with file upload test"
  [11]: https://developer.mozilla.org/en-US/docs/Web/API/FormData "FormData in XMLHttpRequest 2 on MDN"
  [12]: https://github.com/ariya/phantomjs/issues/11013 "new Blob() throws error"
  [13]: https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers.onerror "GlobalEventHandlers.onerror on MDN"
  [14]: https://github.com/ariya/phantomjs/issues/10522 "Function.prototype.bind is undefined"
  [15]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind "Function.prototype.bind on MDN"
  [16]: https://github.com/eligrey/Blob.js "eligrey/Blob.js on GitHub"
  [17]: https://github.com/vvo/selenium-standalone "vvo/selenium-standalone on GitHub"
  [18]: https://github.com/jmreidy/grunt-mocha-webdriver/pull/18 "Run against local selenium instances"
  [19]: https://github.com/bevacqua/grunt-mocha-webdriver-painful "bevacqua/grunt-mocha-webdriver-painful on GitHub"
  [20]: https://github.com/bevacqua/selenium-standalone-painful "bevacqua/selenium-standalone-painful on GitHub"
  [21]: https://github.com/bevacqua/grunt-integration "bevacqua/grunt-integration on GitHub"
  [22]: https://github.com/visionmedia/mocha "visionmedia/mocha on GitHub"
