# A Hard Place

The front-end development community as a whole has put itself in a <mark>hard place</mark>. We've collectively overlooked issues that arise from using tons of purpose-specific tools, **and for good reason**. Other languages and ecosystems are victims of all-encompassing standard libraries, but the web development community takes pride in **not having** that problem. In the past we fell prey to large utility libraries that did just about anything _-- your jQuery, Underscore, etc._

Large libraries had their benefits -- as well as their drawbacks. It was nice being able to drop-in jQuery, do "all the things", and forget about it. As applications grew in size and complexity, *though*, we couldn't just drop any libraries we found on the Internet in our codebases anymore. That'd result in **much-larger-than-necessary** websites, something we didn't want. Thus, over time, the community understood how we could benefit from smaller modules, and started working on micro libraries _-- at a time where most "libraries" were developed as jQuery plugins_. Along came Node.js, and its small modules philosophy turned many of us into firm believers that, **indeed**, writing small modules is the way to go.

Followers of the small module movement started breaking down libraries they built into components. Out came a more modular approach to popular libraries like jQuery and Lodash. True, jQuery had been "modular" for a long time, but the approach they took didn't lend itself to adoption: it relied on consumers utilizing complicated online build tooling where you had to pick what parts of jQuery your application was going to utilize ***-- in advance***. Great in theory, and they certainly could boast about being modular, but not the most user-friendly approach.

> In practice, consumers either used the raw, full jQuery library, or *-- at best --* they took out a few highly irrelevant parts and called it a day. jQuery UI was in a similar place, but had an even weaker position due to the fact that it also depended in jQuery.
>
> When Bower came along, _-- it took a while before jQuery was mirrored onto `npm` as well --_ it became even more obvious that the **custom build wasn't a serious option**, as you'd have to jump through a considerable number of hoops before you could even arrive at a custom bundle.

Lodash took a different path, which favored adoption even though their approach was detrimental to their own build processes _(but not their consumers)_. Starting with v2, they published hundreds of modules to `npm` -- `lodash.find`, `lodash.flatten`, etc. -- where each module represented **one** of the utility functions in Lodash. Later on, starting in v3, they improved upon that and allowed you to pull specific functions as CommonJS modules like `require('lodash/function/bind')`. Even though there's a tree composed of hundreds of small modules in the Lodash codebase, they're still available as a single package on npm. **That is a good thing.**

# Hypermodularization

Around the same time came **hypermodularization**. This is basically the same, in principle, as `lodash` splitting their functionality in hundreds of small modules, bits and pieces of documentation and tests, and `npm` packages. A major difference in an hypermodularized scenario is that you don't see comprehensive distributions anymore. One such example is the original `lodash` package itself, which to this day contains the whole of their utility functions, even though they're now also available as individual pieces. When it comes to `lodash`, you can take the whole thing or a method at a time.

For library authors, hypermodularization makes a lot of sense, as it comes with **a wealth of benefits**.

- You only take what you need
- Reuse functionality across several packages
- Semantic versioning on individual modules _-- not just for packages as a whole_
- Extended documentation and front-facing API surface tests
- Less friction integrating server-side modules in client-side applications

The problem with hypermodularization, though, is that **adoption becomes trickier**. Undoubtedly, people will immediately point at small modules as the culprit. *"Too many API touchpoints"* -- some say. *"Too much plumbing"* -- others point out.  Some might even say that larger bundles of things were better, as you didn't have to spend time forming an opinion as to how half a dozen modules should be plumbed together, or dealing with boilerplate generators such as `yeoman` _(another non-solution -- **code generators are hardly ever the answer**)_.

When I [first ran into tree-shaking][1] I quickly dismissed it as **a nice to have** that would prove hardly more beneficial than [`bundle-collapser`][2], which allows you to save a few hundred bytes in bulky `browserify` bundles. Nice -- sure. Necessary? _Hardly._ Or that's what I thought.

# Tree-shaking is a game breaker

At the time, I misunderstood its use cases. Tree-shaking is a feature available in modern module bundlers _-- [`rollup`][3], namely --_ where ES6 modules are statically analyzed for exports that are being used, and those that are not become left out of the resulting bundle.

Suppose we had the following piece of code:

```js
import _ from 'lodash';
_.keys({ pony: 'foo' });
```

How cool would it be if a compiler would turn that into something like this?

```js
var _ = {
  keys: function (o) {
    return Object.keys(o);
  }
};
_.keys({ pony: 'foo' });
```

> <sub>Disclaimer: Actual `lodash` code is [not as contrived][4], this is merely an illustration.</sub>

I had hoped `rollup` would do that, but it interprets `lodash` as an external dependency, presumably because its written under ES5, or maybe just because the package is in `node_modules`. Nevertheless, if we were able to take code like the previous snippet and turn it into the second -- smaller -- one, we'd **eliminate one of the biggest drawbacks of large distributions** such as the `lodash` package and similar utility libraries: **people**.

People take libraries like `lodash` _-- or jQuery, as we analyzed earlier --_ and insert the whole thing into their codebases. If a simple bundler plugin could deal with getting rid of everything in `lodash` they aren't using, footprint is **one less thing we'd have to worry about**.

The other set of drawbacks in large distributions can't be solved by consumers, and should be resolved by implementers instead. Incidentally, these things are already solved by hypermodularization: **maintability, documentation, ease of contribution, etc.** As modules get smaller, they also become easier to maintain, document, test, and contribute to, lowering the barrier of entry. A large monolith on the other hand usually involves some sort of learning curve, makes people scared of breaking undocumented functionality, and so on.

One drawback when it comes to code contributions and support requests _-- however (and amusingly) --_ is that maintaining an hypermodularized ecosystem is a hassle when they are highly related and kept in several different repositories. Babel has *an excellent document* that outlines [their **<mark>monorepo</mark> culture**][7] and how it has allowed them to contain issues arising from dealing with support requests against their many hypermodular, interconnected components.

# Consolidating Opinions

In a hypermodular ecosystem, it becomes increasingly hard to plumb pieces of code together. In a monolithic ecosystem, it becomes increasingly annoying to deal with large chunks of code that you don't need. _We need to consolidate the two._

**I believe in hypermodular components**. They are great at what they do, they follow the *"do one specific thing very well"* philosophy and are primed to thrive in an open-source community such as ours.

**I never much liked** comprehensive libraries like jQuery in terms of API surface, but a lot of that dislike *goes away* when a tool such as tree-shaking is effectively applied across the board.

Moreover, when we take a look at application logic that is mostly concerned with plumbing discrete libraries in the application level, we begin to see how hypermodular components should be packaged in larger distributions. In one of the opening lines in this article I stated that _"the web community is as opinionated as it is large"_. I opine **we need to become more opinionated** as far as library authorship goes.

[![Screenshot showing code plumbing hypermodular libraries at the application level][5]][6]

<sub>_One such example of plumbing at the application level, extracted from [a Medium article][6]._</sub>

Not only would we be getting rid of obnoxious plumbing, but we'd also steer more users towards best practices, we'd spend less time arguing about what approach is correct, and we'd spend more time being productive in day-to-day development. While flexibility and hyper-modularity are great features, compromising on a set of opinions and reducing noise at the implementation level are similarly honorable objectives to arrive at.

Nothing is to prevent us from keeping the lower-levels of our architectures hypermodularized. This is all good and well -- it is one of the fundamental pillars of modern JavaScript development. It is also true that the crux of our problems today stem not from one library or another being hypermodular, but from the ecosystem as a whole being developed this way.

> In that sense, **we needn't cry** about `babel@6` asking us to install a couple more packages. If you dislike doing that every time, build a wrapper around it with some opinions on top. Do the same for React packages you use and are tired of plumbing over and over in all your applications. Avoid generators with a passion, but **opinionate your way through the ocean of hypermodules** we find ourselves swimming around in.

Write wrappers and intermediate libraries that live outside of your application core while consuming hypermodules. Keep your opinions to those intermediate libraries, while keeping hypermodules discrete. In this sense, you could think of an hypermodule as `lodash/function/bind` and an intermediate library as `lodash`. I use `lodash` as an example because it's one of the best representations out there today of what constitutes an hypermodular library. One that has hundreds of components, but **opinions are not one of them**.

An ecosystem that's founded on intermediate libraries on top of hypermodules could fare much better. The bulk of implementation, testing, and documentation would fall in hypermodules, while opinions and plumbing could be kept in an intermediate layer we are only just yet starting to even consider. Presumably, code in the intermediate layer could be bulkier, but that should be something that _-- with proper and better tooling --_ a feature like tree-shaking could take care of.

[1]: http://www.2ality.com/2015/12/webpack-tree-shaking.html "Tree-shaking with webpack 2 and Babel 6 on 2ality.com"
[2]: https://github.com/substack/bundle-collapser "substack/bundle-collapser on GitHub"
[3]: https://github.com/rollup/rollup "rollup/rollup on GitHub"
[4]: https://github.com/lodash/lodash/blob/6889837d2d3f506ef36fd5cf7368b0ad70c7431f/lodash.js#L11122-L11140 "_.keys method in the lodash codebase"
[5]: https://i.imgur.com/4ArrsqC.png
[6]: https://medium.com/@ericclemmons/javascript-fatigue-48d4011b6fc4 "Javascript Fatigue on Medium"
[7]: https://github.com/babel/babel/blob/master/doc/design/monorepo.md "Why is Babel a monorepo?"
