# Taming Asynchronous JavaScript #

Last month, a series of _very_ interesting articles regarding **async coding style**, in Node, popped up. The discussion spanned a few more subjects than _just coding style_, it was analyzed in a more _theoretical level_, as well as a _deep technical level_, obviously on the _practical level_, and _even politics_ saw the limelight in this _fascinating_ argument.

I'll try my best to put together a coherent article on the subject that doesn't put you to sleep. But I wouldn't bet my horse on that.

I'd bet _yours_, though.

I do realize I got _one month too late_ to the party, but [like I mentioned](/the-architecture-of-productivity "The Architecture of Productivity"), I was too busy visiting coffee shops in Amsterdam. Without further ado...

### Case Study: [Callbacks](https://github.com/caolan/async "'async' callback library") vs. [Promises](https://github.com/kriskowal/q "'q' promise library") ###

Firstly, I would like to cite the series of resources that prompted this blog post.

- [You're Missing the Point of Promises](http://domenic.me/2012/10/14/youre-missing-the-point-of-promises "An introduction to promises"), an introduction to promises
- [Node's biggest missed opportunity](http://blog.jcoglan.com/2013/03/30/callbacks-are-imperative-promises-are-functional-nodes-biggest-missed-opportunity "James' post in favor of promises in Node"), the article that triggered the _blog-post hell_, by [James Coglan](https://github.com/jcoglan "James Coglan on GitHub")
- [Broken Promises](http://www.futurealoof.com/posts/broken-promises.html "Broken Promises, written by Mikeal Rogers"), narrating why promises were `.reject()`'d in Node, by [Mikeal Rogers](https://github.com/mikeal "Mikeal Rogers on GitHub")
- [Broken Promises](http://sealedabstract.com/code/broken-promises "Broken Promises, written by Drew Crawford, an iOS developer"), another reply, depicting the pitfalls of promises and evangelizing patterns
- [Callbacks, promises, and simplicity](http://blog.jcoglan.com/2013/04/01/callbacks-promises-and-simplicity "James' conclusions on the matter"), where James makes some concessions and concludes his point

These are possibly the lengthiest articles written on the subject, but they definitely are the most interesting ones. I recommend to take your time and read through all of them, you won't regret it.

[...]

So, you've read them all? **Great!**

Lets back up a little bit, now, and point out the source of the conflict. Node _supported promises_ early on, but they decided to [drop support](https://groups.google.com/forum/?fromgroups=#!msg/nodejs/sWE0Oa80iNg/-n7xPyOdGd8J "Ryan explains why promises aren't such a good fit for Node") for them shortly afterwards, partially because of the volume of disagreement they generated. Thus, they _sticked with callbacks_.

![call-me.jpg][1]

The different modules packaged through <kbd>npm</kbd>, swiftly followed the convention of passing `function(err, results){}` as the last parameter, a convention was silently born.

# My Point of View #

While developing [this blogging engine](https://github.com/bevacqua/ponyfoo "ponyfoo on GitHub"), and _particularly_ the [asset manager](https://github.com/bevacqua/node-assetify "assetify on GitHub") behind it, I made extensive use of [async](https://github.com/caolan/async "async on GitHub"), which vastly improves upon the _raw power_ of manually chaining callbacks.

I felt comfortable dealing with callbacks, but some more _complex scenarios demanded a more robust solution_, such as the ones **async** provides.

As far as promises go, I've been working with [jQuery.Deferred](http://api.jquery.com/category/deferred-object/ "Deferred Object - jQuery API docs") for a while now, mostly in AJAX scenarios, though.

Recently, I started working on an application that makes more extensive use of promises, both through [AngularJS](http://angularjs.org/ "AngularJS MVW Framework") on the client-side, and using [Q](https://github.com/kriskowal/q "'q' promise library") on the server-side.

On the client-side, this makes perfect sense. We have **jQuery** performing similar operations, and the complexity requirements for asynchronous code aren't as steep as they are in the back-end.

Front-end async operations _tend to be way simpler_ than those performed on the server-side. There _rarely_ is the need to create a deeply nested hierarchy of callbacks: an event handler making an AJAX request and maybe even doing something else with the response. You might even have a thin layer that creates a cache of the responses, but that's about it. From this standpoint, it makes sense using promises.

On the other side, promises tend to result in more convoluted, complex code in Node. 

![broken-promises.jpg][2]

> To me, it's not a matter of simplicity, [as James puts it](http://blog.jcoglan.com/2013/04/01/callbacks-promises-and-simplicity "Callbacks, promises, and simplicity"), but rather, a matter of **both practicality and productivity**.

> Practicality, because it's just _more straightforward_ to use callbacks in an environment where **it's the standard to do so**.
_Productivity_ is a side-effect bonus here, bending the application to use promises in a portion of the code would be an unwarranted waste of time.

And _think of the drawbacks_. It would also be unnatural and lead to a chaotic codebase where everyone has an opinion on how code should look like. It's not even that promises are _wrong_, it's just that, _at this point in time_, they **don't belong** in Node.

Again, not because promises are _wrong_, or _"broken"_, but they don't fit in, they don't accomplish anything that can't be done using callbacks. Besides, mixing both styles might even _complicate_ matters, making it particularly confusing to handle errors, for example.

## Error Handling Conventions in JavaScript ##

There is a very simple convention in JavaScript you should abide by, if you want to build _successful and clean_ applications. And that is, how to write properly error-handling functions.

> Synchronous code should `throw`, Asynchronous code shouldn't. Ever.

```js
function sync(){
    try{
        return taskThatMayThrow();
    }catch(e){
        console.log(e); // handle
    }
}

function async(done){
    operation(function(err, result){
        if(err){
            return done(err); // bubble up
        }
        result += 1; // [...] further processing
        done(null, result);
    });
}
```

Simple enough, when dealing with asynchronous code, you should always _bubble exceptions_ up, so that _the caller can choose_ to either handle them, or bubble them further up the callback chain. This way, your application can handle errors gracefully, rather than quit unexpectedly (or require `process.on('uncaughtException')` in order to survive).

### Conclusion ###

I definitely think Node is better off without promises, broken or otherwise. The client-side ecosystem is a little less well-defined, and as such, a more suitable home for promises, but even then, it's just a matter of preference to use one or the other.

  [1]: http://i.imgur.com/1B12xqM.jpg "Overly attached callbacks"
  [2]: http://i.imgur.com/qPrHNz6.jpg "Broken Promises"
