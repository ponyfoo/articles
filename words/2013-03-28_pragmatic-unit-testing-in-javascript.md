# Pragmatic Unit Testing in JavaScript #

More often than not, companies completely (and _irresponsibly_) disregard JavaScript as code that _should be unit tested_. They might test their back-end code, it may be in C#, Ruby, Java, or even PHP, or _just about any other language_. But there's a good chance that the front-end code is **thoroughly untested**.

_Integration level testing_ with tools such as [Selenium](http://docs.seleniumhq.org/ "Web Browser Automation Testing") is nice in theory, but way too impractical (you have to set up a server), and particularly slow (loading browsers and computing the recorded actions takes its toll). As such it's rarely part of build processes, and it's run manually (with a single command, but manually nonetheless).

So why is that JavaScript gets _treated so differently_ from the rest of languages?

![js-discrimination.jpg][1]

# Arguments against testing JavaScript in the client-side #

- JavaScript, and _the web_ in general are [_tremendously_ fault tolerant](http://www.codinghorror.com/blog/2007/04/javascript-and-html-forgiveness-by-default.html "JavaScript and HTML: Forgiveness by Default")
- Errors in front-end code are _not perceived to be as **impactful** as back-end errors_. Since this is client-side code, no sensitive data will get lost or _accidentally deleted_. As long as the back-end is safe, data is safe
- JavaScript is _challenging to test_

While it's true that errors in the front-end are not _as likely_ as permeate a robust back-end layer and cause trouble, there are real threats out there, such as [XSS attacks](http://en.wikipedia.org/wiki/Cross-site_scripting "Cross-site scripting"), which are enabled by front and back-end alike.

## Why is testing JavaScript _hard_? ##

I could give a list of reasons why testing JavaScript is hard, but _it all boils down to it being a dynamic language_. There's no compiler. Sometimes that it great, we've come to _love_ the language for its dynamic nature. However, it makes testing _harder_.

As such, our first line of defense should be [linting](/2013/03/22/managing-code-quality-in-nodejs "Managing Code Quality in NodeJS"). This is the closest we have to a compiler, in terms of assurance that _our code won't break_.

But that obviously isn't _enough_. Linting is _just the first step_ in the right direction. Our code should be tested. 

## Back to the dynamic nature of JavaScript ##

I think another important factor in testing is _visbility_. In statically typed languages such as C#, variables can be `private`, `public`, `protected`, `internal`, or some combination of those. In JS it's either `private` or `public`.

There are no _statically defined interfaces_. You might be used to interfaces such as:

```cs
public interface ITrackable
{
    int TrackingNumber { get; }
    void Track();
    bool Untrack();
    bool IsBeingTracked { get; }
}
```

Bear with me for this small example of a testable class, written in C#:

```cs
public class Testable
{
    private readonly ITrackable _trackable;
    
    public Testable(ITrackable trackable)
    {
        _trackable = trackable;
    }
    
    public bool CallMeTracy()
    {
        _trackable.Track();
        return true;
    }
}
```

The code doesn't make any sense. I know. The case in point is that, using [Dependency Injection](http://www.amazon.com/dp/1935182501 "Dependency Injection in .NET"), `Testable` becomes very easily testable. Here's a sample test:

```cs
[TestCase]
public class TestableTests
{
    private Testable testable;
    
    [SetUp]
    public void Setup()
    {
        var mock = new FakeTrackable();
        testable = new Testable(mock);
    }
    
    [Test]
    public void should_call_me_tracy_and_return_true()
    {
        bool result = testable.CallMeTracy();
        
        Assert.IsTrue(result, "Expected CallMeTracy to return true.");
    }
}

public class FakeTrackable : ITrackable
{
    public void Track()
    {
    }

    // ... other implementation stubs ...
}
```

How do we reach a similar state of affairs in JavaScript? We simply can't. We must _adapt to the dynamism_, embrace it.
    
# How to test, then? #

It's not as bad as you might be thinking right now. It's just a matter of changing the way you think about testing.

JavaScript testing needs to be _even more thorough_. Your code can't statically provide the interface you desire? Then _write tests_ to ensure it exposes that interface. Welcome to **TDD**!

Lets refer to a similar example in JavaScript

```js
function Testable(trackable){
    this.callMeTracy = function(){
        trackable.track();
        return true;
    };
}
```

And the test, which might be something like this

```js
describe('Testable', function(){
    it('should return true when calling him Tracy', function(){
        var him = new Testable({
            track: function(){
                // this is just a mock
            }
        });
        
        expect(him.callMeTracy).toBeDefined();
        expect(him.callMeTracy()).toBeTruthy();
    });
});
```
    
Obviously, a clear difference here is that `trackable` can be _anything_, unless it's constrained by _guard clauses_. But, that's generally something that JavaScript developers shy away from, given the _sheer power_ it provides.

## Dependency Injection in JavaScript ##

There's a catch though, injecting dependencies in JavaScript is _kind of a mess_. 

If you are writing tests for **Node.JS** code, you might be in luck. I have been using [proxyquire](https://github.com/thlorenz/proxyquire "thlorenz/proxyquire on GitHub"), which basically allows you to, _without modifying a single line of source code_, test modules and mock dependencies on other modules loaded with `require`. It requires some setting up, but it made me pretty happy thus far.

**Browser code** is _a different story_, it often follows patterns similar to this:

```js
!function(window, $){
    window.myThing = {
        annoy: function(){
            alert('about to become very annoying!');
            $('a, span, b, em').wrap('<marquee/>');
        }
    };
}(window, jQuery);
```

This kind of code is indeed hard to test, but you could always just load the affected JS file in isolation, after creating stubs for the global objects you need. An example would be:

```
var window = {},
    jQuery = function(){
        return {
            wrap: function(){
            }
        };
    };
```
        
Once you get tired of helplessly mocking your way out of trouble, you should use a real stubbing and mocking framework, such as [Sinon.JS](http://sinonjs.org/ "Standalone test spies, stubs and mocks"). Also, remember that using spies to verify that callbacks (such as `callMeTracy`, and `wrap`) are invoked, and passed _the correct parameters_!

## Unit Testing Frameworks ##

I like [Jasmine](http://pivotal.github.com/jasmine "Jasmine BDD framework for testing") for my unit testing, but there are plenty of frameworks to choose from. [Mocha](http://visionmedia.github.com/mocha "Mocha test framework") is another popular one.

Once you realize that testing in JavaScript is _not that bad_, and look at it from _another perspective_, you can start exploiting the very dynamic nature you feared to help you build _even better tests_!

A pattern I commonly use when defining unit tests in Jasmine, is to prepare a list of test cases (expected input and output), and then run them all at once. Here's an example taken directly from [one of my GitHub repositories](https://github.com/bevacqua/jsn/blob/29384246d28d688475669375423568c54439feed/test/spec/text.js "jsn GitHub repository"):

```js
describe('test cases', function(){
    var cases = [],
        context = {
            plain: 'plain',
            foo: {
                bar: 'baz',
                undef: undefined,
                nil: null,
                num: 12
            },
            color: 'red',
            how: { awesome: 'very' }
        };

    function include(input,output){ cases.push({ input: input, output: output }); }

    include('@plain','plain');
    include('@foo.bar','baz');
    include('@foo.undef',undefined);
    include('@foo.nil',null);
    include('@foo.num',12);
    include('@@foo.bar','@foo.bar');

    cases.forEach(function(testCase,i){
        var replace = text.replace(Object.create(context));

        it('should return expected output for case #' + (i+1), function(){
            expect(replace(testCase.input)).toEqual(testCase.output);
        });
    });
});
```
    
This is something that _simply cannot be accomplished statically_. You could get here with reflection, but it's just unnatural to static languages.


  [1]: http://i.imgur.com/WUHDGPw.jpg "JavaScript statements are people too!"