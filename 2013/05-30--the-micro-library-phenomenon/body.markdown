## ThinDOM ##

The first example that pops to mind is [ThinDOM](https://github.com/jacobgreenleaf/ThinDOM "ThinDOM by imgur, on GitHub"), as it only came to light a little over [a week ago](http://imgur.com/blog/2013/05/21/tech-tuesday-jquery-dom-performance/ "jQuery DOM performance - imgur blog").

ThinDOM is a thin **DOM** wrapper out-classes [jQuery](https://github.com/jquery/jquery "jQuery on GitHub") when it comes to **DOM** manipulation. It provides only _a few methods_, which are conveniently named like their jQuery counterparts:

- `.append()` is a blisteringly fast alternative to `$.append`
- `.css()` doesn't do any validation, or value transformation, [like $.css does](https://github.com/jquery/jquery/blob/master/src/css.js#L111-L132 "$.css source on GitHub")
- `.html()` doesn't provide any safety, no parsing, nothing. [$.html](https://github.com/jquery/jquery/blob/master/src/manipulation.js#L124-L161 "$.html source on GitHub") is a tad slower
- `.attr()` just sets attribute values. That's it. Here's [$.attr](https://github.com/jquery/jquery/blob/master/src/attributes.js#L288-L334 "$.attr source on GitHub")'s take
- `.get()` unwraps the element wrapped under **ThinDOM**

Their API is _kind of clunky_. Their example isn't the prettiest.

```js
var captionDOM = new ThinDOM('div').attr('class', 'caption')
    .append(new ThinDOM('div').attr('class', 'votes')
            .append(new ThinDOM('a').attr({'class': 'up', 'href': '#'}))
            .append(new ThinDOM('a').attr({'class': 'down', 'href': '#'})))
    .append(new ThinDOM('div').attr('class', 'meta')
            .append(new ThinDOM('span').text(author + ' - '))
            .append(new ThinDOM('span').text(points + ' point' + plural)))
    .append(new ThinDOM('p').html(body)).get();
```

But... if _it's performance_ you need, then it definitely wins out.

## Zepto ##

This library started out as an alternative to jQuery for the _mobile browser_. They didn't need to support IE in mobile browsers, so they cut that from what jQuery offers. To further expand this _footprint gap_, you can choose to leave out the sub-modules you don't need, in order to make the footprint _even lighter_.

jQuery is currently sporting a **32kB** footprint, [Zepto](http://zeptojs.com/ "ZeptoJS lightweight jQuery alternative") is a lightweight alternative, currently sitting at a maximum of **9.7kB** minified and gzipped.

Zepto offers **no support for IE**, they recommend you to _fall back to jQuery_ in IE.

In conclusion, Zepto is awesome. If you can get away with it.

There are thousands of micro-libraries like these, and probably even more so in the world of Node and [npm](npmjs.org "Node Package Manager").

# Why is any of this relevant? #

It's about philosophy. _Design philosophy_. And I think we owe this, in part, to Node. Most [npm](npmjs.org "Node Package Manager") modules are very compact, determined to fill that little hole and become the **de-facto tool** for that _super-specific purpose_.

> **All of them are open-source**. More people should realize how **huge** that is. It's huge because it means you can learn how the best, successfull frameworks and libraries do it. It's true that open-source is gaining a lot of traction. Ever since **GitHub** came out in 2008, and then **Node** in 2009.
> In the world of JavaScript, the concept of closed-source is something that you can't even begin to fathom, and that _encourages_ open-source.

Since we were talking about imgur, let's look at the [search results in npm for the keyword "imgur"](https://npmjs.org/search?q=imgur "npm search results"). There are _around ten_ imgur related modules. Many of these, _just upload_ to imgur. Such is the case of [node-imgur](https://github.com/kaimallea/node-imgur "node-imgur on GitHub").

Sure, micro comes with _a cost_. Everyone wants to write their own modules. Generally speaking, though, the best ones prevail. The others fall off into oblivion.

In the end, having many alternative frameworks that do the same thing, gives you **choice**, albeit _a tough one_ sometimes. But, I'd much rather have to pick from a ton of micro-frameworks that, potentially, do what I want. It beats having to use a _giant library_ that can do everything but it's **exaggeratedly complicated**, or heavily undocumented.

## Identifying the Source ##

Granted, taking your pick can be [tedious at first](/2013/01/18/asset-management-in-node "Asset Management in Node"). But if you are not content with what the existing solutions do, you can always [roll out your own](/2013/01/23/publishing-nodejs-packages-with-npm "Publishing Node.JS packages with npm"), and help the next guy in the process.

When I first wrote about rolling my own packages, I was just getting started in the world of Node, and I considered the **vastity** of choice _an issue_. I changed my mind about that.

Micro-frameworks in JavaScript are a phenomenon that we hardly ever see in other communities, and therefore, it's a hard thing to wrap our head around.

We have tons of packages for handling parallelism in regular code, not just for ocassional and complicated multi-threaded services. But it doesn't just end with parallelism. We have utility libraries for just about anything. And we owe a large portion of that to the fact that Node is largely unopinionated. 

> **Node barely does anything**. It just provides a _layer of abstraction_ over the operating system. Other than that, you'll probably have to rely in someone's package, or write your own.

Some packages are [absurdly specific](https://github.com/bminer/node-static-asset "static-asset on GitHub"). Most other communities _don't even bother_ to create modules that small.

> It's like the [npm](npmjs.org "Node Package Manager") community discovered _a new sub-atomic particle_!
