# A Brief Rant About _Mocha_ et al

[Mocha coffee][1] is espresso mixed with hot milk, but _with added chocolate_. Now, I know [what you must be thinking][2], but this doesn't have anything to do with coffee. Rather, it's all about the [mocha][3]. _Mocha ain't that good for you either._ Suppose you are considering to implement tests in that project of yours at work. _\*Finally!\*_, you can hear your superiors whispering. At long last will your project be validated by a battery of tests that can be executed on every build. We'll be able to [fail fast][4] by stopping deployments on their tracks _whenever a single test fails_. Our production systems won't crash as often _(hopefully **never** at all)_, and code quality overall will improve a thousandfold!

There's plenty of testing suites out there, large and small, but in this article we'll have to focus on the small ones, the unit testing libraries of the JavaScript universe. You have a choice to make. You need to pick the systems that will define your application's quality standard from here on out.

Faced with this choice, should you pick a framework like Mocha? Let's see.

```js
var should = require('should');
var personCtrl = require('../controllers/person');

describe('personCtrl', function () {
  it('is literally littered with globals everywhere', function () {
    should(personCtrl.mouth).not.throw(Up);
  });
});
```

Mocha has plenty of globals like `describe`, `it`, `beforeEach`, `before`, `after`. Evidently, they couldn't make up their mind with regards to assertion flavors, because testers _love_ [BDD][5] while developers often prefer _precise assertions_. It's up to you to pick the assertion engine, you do get the globals though! There's no question about _that_! One unspoken problem about globals is that they make it that much harder to infer the API for a library just _from usage alone_. If the API is contained behind a module-wall like `require('assert')`, you can at least investigate: `keys(assert)` gives you the API methods _(use `Object.keys` outside of the browser console)_, and you can do some blind poking. When the API is **a fiery global-spilling ball of death**, it's much harder to tell. Sure, you can poke around the `global` object instead, but why go through the trouble?

> If `describe`, `before` and `it` are globals, **who knows** what else is a global? Or why the hell `assert` is _not one of them_?

Setting aside the fact that `should` extends the `Object` prototype, there's the whole _"test harness"_ thing. You need to take some **densely specific steps** in order to be able to actually execute your tests. Want tests to run in the server and the browser? You're going to have _a hell of a time!_

![Mocha test runner example][6]

<sub>_Running tests via the Mocha test harness in a single, succint command._</sub>

That's enough caffè mocha for the day. Let's shift our attention to a pleasant test engine that isn't actually _detrimental_ to the overall code quality in your applications.

# Introducing Tape

It's much better to go for a modular testing library, and [tape][7] fits the bill, so that's what I've been using for the better part of the last few years.

```js
var test = require('tape');
var personCtrl = require('../controllers/person');

test('we are the new anti-globalization protest front', function (t) {
  t.doesNotThrow(function () {
    personCtrl.mouth();
  });
  t.end();
});
```

Needless to say, you don't _have_ to use `tape`'s built in assertions. You could use `should` _(more like shouldn't)_, but at least you get _a default_ assertion engine. The `describe` aspect of the **de-facto thousand-line long test file** is translated into making smaller test files that are named just like the module you intend to unit test. `it` becomes `test`, and the only awkward piece of the puzzle is naming `require('tape')` `test` instead of `tape`.

Of course, you could always just name it `tape` if that weirds you out.

#### How is Tape better?

Besides not endorsing `global` warming, there's a few other differences in how Tape operates. One of them, is how you define your tests. You may have noted that I used `t.end` in my example. That method is used to signal that a test method has ended. If your test is asynchronous, simply call `t.end` in your callback.

```js
var test = require('tape');
var personCtrl = require('../controllers/person');

test('munching chews 50 times before swallowing a piece of food', function (t) {
  personCtrl.munch(function munched (err, result) {
    t.equal(err, null);
    t.equal(result.timing.pieces, 1);
    t.equal(result.timing.chewing, 50);
    t.end();
  });
});
```

Another approach to setting `t.end` might be using `t.plan(count)`. Plan lets you specify how many assertions you expect your test to execute. This comes in handy sometimes, and can double as an extra assertion where you verify that all the other assertions are executed. Note that the test will fail if the count is off even by one, meaning that most of the time going for simply `t.end` will save you the hassle of updating `t.plan` statements with the correct assertion count. That being said, `t.plan` is a useful _"guard clause"_ when you're unsure whether a callback _(presumably containing much needed assertions)_ will run or not.

Tape allows you to quickly turn off tests by changing `test` statements into `test.skip`, which is something that you can do with most test engines.

I've always been baffled about `before`, `afterEach`, and friends. I presume these were copied from mature test frameworks in other languages, but they're largely useless in the dynamic prairie that is JavaScript. 

You could just replicate those things with a few lines of code, _for example:_

```js
function wrapper (description, fn) {
  test(description, function (t) {
    setup();
    fn(t);
    teardown();
  });
}
```

Then you could just use `wrapper` instead of `test` for those tests that you want to prepare and then dispose of. This is more transparent than just invoking the `beforeEach` global method inside the `before` and `it` globals and hoping for the best.

Given that this is just code and there aren't mystifying globals laying around, you could _easily come up with composable modules_ that are able to, for example, [create a MongoDB database and drop it][13] before and after every single test, providing the necessary isolation to individual integration tests.

#### Look ma, no harness!

Once you're done writing your tests, you can simply execute them using `node`, _no harness or test runners involved_, imagine that!

```shell
node test/*.js
```

If you need to run the tests on a browser you could use [testling][10] after browserifying the test files.

```shell
browserify test/*.js | testling
```

Testling also offers the ability to run the tests [in many different browsers][11] by configuring the `testling` field in your `package.json` file.

#### Tap Out Reporters

Tape produces `TAP` output. [TAP][8] is a well-defined format that _kind of looks like this:_

```
1..9
ok 1 concurrent() should return the results as expected
ok 2 map() should return the results as expected
ok 3 waterfall() should run tasks in a waterfall
ok 4 series() should run tasks in a series as array
ok 5 series() should run tasks in a series as object
ok 6 series() should short-circuit on error
ok 7 concurrent() should run tasks concurrently as array
ok 8 concurrent() should run tasks concurrently as object
ok 9 concurrent() should short-circuit on error
# tests 9
# pass 9
# fail 0
1..1
ok 1 Test was run
#TAP meta information
0 errors
```

Just like Mocha, Tape provides you with a bunch of [reporters that display fancier output][9] than machine-readable TAP. One of the benefits of the protocol is that you could use these reporters with anything that produces TAP output, not just `tape`.

# Leveraging `proxyquire` to mock modules

Besides using test databases, you could also mock services and libraries you don't want to have an influence on the outcome of a particular test. There are plenty of ways of doing this, but I find that the best of them is to grab the dependency by its roots and remove it altogether. The [proxyquire][14] module makes the process quite easy for both the server and the browser.

There are some [samples on how to use proxyquire][15] in my book. Suppose you have a module `./timesTen` like the one below:

```js
var multiply = require('./multiply');

function timesTen (input) {
  return multiply(input, 10);
}

module.exports = timesTen;
```

Suppose also that you want to create some tests for `./timesTen`, but you've already taken care of tests for `./multiply` in another test file. Thus, you can mock `./multiply` using `proxyquire` in your tests.

```js
var proxyquire = require('proxyquire');
var test = require('tape');

test('timesTen returns output from multiply', function (t) {
  // arrange
  var multiplyStub = function (input) {
    return Infinity;
  };
  var timesTen = proxyquire('../timesTen', {
    './multiply': multiplyStub
  });
  
  // act
  var result = timesTen(20);

  // assert
  t.equal(result, Infinity);
  t.end();
});
```

Of course, that's **a horrendously simplistic way of testing** something out, but it was also a very contrived example. In the real world you probably want to mock up a service that ends up accessing a database, **Amazon S3**, the _Imgur API_, or some other external service you really don't want affecting the outcome of your tests. If we're talking about the client-side, even better. You can use `proxyquire` to hide away interaction with the _DOM_, _XHR_, _WebSockets_, _or anything_ that's not pertinent to the test case.

# Complementing `proxyquire` with Sinon.JS

[Sinon.JS][16] provides you with many ways in which you can mock objects in JavaScript. While `proxyquire` is the best thing you could be using to replace an entire module with a stub, `sinon` makes it quite easy to _create_ the stubs that you'll be _providing_ to `proxyquire`.

For example, suppose a method you want to mock within a module is supposed to end up calling `done(null, user)`. Using `sinon`, that becomes:

```js
var sinon = require('sinon');
var user = { username: 'ponyfoo', id: 1234 };
var getUser = sinon.stub().yields(null, user);
var userService = {
  getUser: getUser
};
```

Now you can assert that the user is being properly utilized by comparing whatever you get back to the `user` mock you've defined that `getUser` should yield. Assuming you're consuming the user service from within a controller, the snippet below could very well be a unit test for the controller, where you want to ensure that the response doesn't contain the user's `id` field.

```js
var proxyquire = require('proxyquire');
var userController = proxyquire('../controllers/user', {
  '../services/user': userService
});
userController.prepareModel(function (err, model) {
  t.equal(err, null);
  t.equal(model.username, 'ponyfoo');
  t.notOk('id' in model);
  t.end();
});
```

A user controller that passes the test _might look something like the following piece of code._

```js
var userService = require('../services/user');

function prepareModel (done) {
  userService.getUser(1234, function (err, user) {
    if (err) {
      done(err); return;
    }
    done(null, { username: user.username });
  });
}

module.exports = {
  prepareModel: prepareModel
};
```

Just like it provides an excellent stubbing facility, `sinon` also allows you to spy on callbacks.

# Using `sinon` to spy on callbacks

You can create a method that's able to tell you how many times it was called, what arguments were used each time, and so on. Suppose that we just want to make sure the user controller calls `userService.getUser` at all. We can write a test like so:

```js
var test = require('tape');
var sinon = require('sinon');
var proxyquire = require('proxyquire');

test('userController calls userService.getUser', function (t) {
  var getUser = sinon.spy();
  var userController = proxyquire('../controllers/user', {
    '../services/user': { getUser: getUser }
  });

  userController.prepareModel(function () {});

  t.equal(getUser.callCount, 1);
  t.end();
});
```

Just making sure the method was called once might not be enough, but you can also verify the precise arguments that were used.

```js
t.equal(getUser.firstCall.args[0], 1234);
t.equal(typeof getUser.firstCall.args[1], 'function');
t.end();
```

At that point you might go the extra mile and invoke the callback yourself. Effectively setting up a _"step-through"_ right in your test case, as you can control the exact timing with which your callbacks get invoked.

```js
var user = { username: 'ponyfoo', id: 1234 };
getUser.firstCall.args[1](null, user);
```

All together now:

```js
var test = require('tape');
var sinon = require('sinon');
var proxyquire = require('proxyquire');

test('userController calls userService.getUser', function (t) {
  var user = { username: 'ponyfoo', id: 1234 };
  var getUser = sinon.spy();
  var userController = proxyquire('../controllers/user', {
    '../services/user': { getUser: getUser }
  });

  userController.prepareModel(wrapUp);

  t.equal(getUser.firstCall.args[0], 1234);
  t.equal(typeof getUser.firstCall.args[1], 'function');

  getUser.firstCall.args[1](null, user);

  function wrapUp (err, model) {
    t.equal(err, null);
    t.equal(model.username, 'ponyfoo');
    t.notOk('id' in model);
    t.end();
  }
});
```

Using this technique you could make assertions about the callback that gets passed in `userController` to the `userService.getUser` method. Of course, this is not always necessary, as you typically want to be testing inputs against outputs, and _never implementation details_.

That way, if the underlying implementation changes, you won't be invalidating tens of ineffective tests. As long as the inputs match the expected outputs, all should be well.

# Conclusions

I realize that frameworks like Jasmine were **probably built with testers in mind**. You know, the kind of people who favor BDD and [the Cucumber plain text human-readable DSL][12]. More modular alternatives should be favored by developers.

The benefits of modular test frameworks like `tape` far outweight those of global-sprinklers such as Jasmine or Mocha, when it comes to developers writing maintainable tests. Regardless of your choice, `sinon` is an obvious inclusion in your test suites, as is `proxyquire` if you're dealing with CommonJS modules.

[1]: https://en.wikipedia.org/wiki/Caff%C3%A8_mocha "Caffè Mocha on Wikipedia"
[2]: /articles/we-dont-want-your-coffee "We don't want your Coffee"
[3]: https://github.com/mochajs/mocha "mochajs/mocha on GitHub"
[4]: http://martinfowler.com/ieeeSoftware/failFast.pdf "A random 'Fail Fast' article"
[5]: https://en.wikipedia.org/wiki/Behavior-driven_development "Behavior-driven Development"
[6]: https://i.imgur.com/KLESZQN.png "Screen Shot 2015-07-08 at 12.01.04.png"
[7]: https://github.com/substack/tape "substack/tape on GitHub"
[8]: https://testanything.org/ "TAP: Test Anything Protocol"
[9]: https://github.com/substack/tape#pretty-reporters "Pretty reporters for Tape"
[10]: https://github.com/substack/testling "substack/testling on GitHub"
[11]: https://github.com/substack/testling/blob/master/doc/testling_field.markdown "Testling package.json configuration documentation"
[12]: https://cucumber.io/ "Cucumber framework"
[13]: http://github.com/bevacqua/mongotape "bevacqua/mongotape on GitHub"
[14]: https://github.com/thlorenz/proxyquire "thlorenz/proxyquire on GitHub"
[15]: https://github.com/buildfirst/buildfirst/tree/master/ch08/05_proxying-your-dependencies "Proxying Your Dependencies, Chapter 8, JavaScript Application Design"
[16]: http://sinonjs.org/ "Standalone test spies, stubs and mocks for JavaScript"
