# Asset management in Node #

Client-side asset management reveals a series of issues with **Node.JS** and **npm** today:

- Node.JS is just a few years old (it was born in _2009_)
- Some areas, such as client-side asset managers, don't have a well defined **best library**
- Publishing packages on [npm](https://npmjs.org/ "npmjs.org") is astonishingly easy
- Large amounts of packages are available

This wouldn't be an issue if **npm** made it possible to _visualize_ which packages are _most actively_ being developed, maintained, and **widely used**. When looking for an asset manager, I found out there were _at least_ five different asset managers I just _had_ to try. But their interfaces felt clunky enough I ended up deciding to contribute to the mess with [yet another asset manager package](https://npmjs.org/package/assetify "assetify on npm").

# Features #

I wanted an asset manager that was:

- **Easy to configure**, I didn't want to spend _hours_ figuring out what the package source was doing, or _supposed_ to be doing
- I wanted to be able to create different **profiles**, yet manage everything the exact same way
- I wanted it to support **pre-processing**, such as compiling _SASS_ or _CoffeeScript_
- **Extensible**, so that if it didn't support something out of the box, I could extend it intuitively to get the behavior I expected
- **Bundling and minification** was also a must, but I wanted to be able to control this.

And besides the above, I wanted the package to also deal with rendering the `<script>` and `<link>` tags. And I wanted it to do this in the _same order_ I passed to the asset manager configuration, since **order matters**.

# Introducing [node-assetify](https://github.com/bevacqua/node-assetify "node-assetify on GitHub") #

I attempted to cover most of **assetify**'s functionality in its [documentation](https://github.com/bevacqua/node-assetify/blob/master/README.md "node-assetify documentation") on **GitHub**.

I think I did a good job of keeping the interface assetify exposes clean and simple. It supports a similar API on the server-side, through the `require('assetify')` module as it does on view contexts, through `res.locals.assetify`.

I built it in such a way that allows you to switch between _development_ and _production_ modes just by turning a boolean, something I couldn't accomplish with the packages I've tried out.

My most recent addition to it (currently version **0.0.8**) was **dynamic asset management**, which, even though I didn't need for [Pony Foo](https://github.com/bevacqua/ponyfoo), I felt this was a feature that couldn't be missing in **assetify**.

A useful pattern in web architecture is to decompose views into smaller chunks, or _partial views_. One of the _drawbacks_ of such modularization, is handling script blocks. If you're _a purist_, you'll irrevocably want your script blocks **grouped together at the bottom of the page**. But you'll also want to _declare them in the same partial view_ where you are going to be needing them.

### Dynamic assets to the rescue ###

With assetify, you can use `res.locals.assetify.js.add`, or just `assetify.js.add` in view contexts, to keep your partials tidy and your scripts grouped. This method is just a glorified way of pushing the source code passed to the `add` function into an array in the request object. Then, when the time comes to `.emit()` your script blocks, dynamic asset blocks will be emitted as well, right after compiled static assets, where they _belong_.