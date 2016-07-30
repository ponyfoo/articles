My usual approach is that there is a module I want -- some piece of functionality I need. If I'm working in an application I'll end up adding a little more code as if I already had a module that did this. For instance, suppose I'm writing an application that automates the process of eating tacos, and I need to add some sauce. I think up an API for the `sauce-adder` module first, and then write up a quick implementation.

Usually I favor `options` objects unless the likelyhood of needing anything more than a single "option" is low. Similarly, I favor providing the "context" as a first argument, unless there's a callback, in which case I try to keep things in just a couple of arguments like `(options, done)`. Returning the `taco` might be good for functional programming and chaning in general, so we could do that as well.

```js
function add (taco, options) {
  if (options.spiciness) {
    taco.spice += Math.pow(2, options.spiciness);
  }
  return taco;
}

module.exports = {
  add: add
};
```

As soon as I'm done writing down a reasonable implementation for a first release, I add what documentation I can. It doesn't need to be perfect, it just needs to exist. I can always improve, iterate on it. I also tend to add a `changelog.md` file like this:

```markdown
# 1.0.0 IPO

- Initial Public Release
```

That helps because whenever I actually have to write up some changes, I'll have the changelog already.

# Documentation in Open-Source

I feel like the single most influential factor driving the popularity of open-source projects is documentation. Without thorough, well-written documentation, consumers are at a complete loss as to how to use a library. I'll take a well-documented library over a slightly more performant one, any day of the week.

There is this cargo cult myth in software engineering where documentation becomes stale the second you finish writing it. That may have been true back when you had to print manuals for desktop software that was then updated with diskettes every few months, but it's definitely `false` in the case of well-maintained open-source projects. In open-source, people seldom merge pull requests without documentation, unless they're willing to update the documentation themselves. In open-source, people understand that documentation is what drives usage, because a well-documented API is an API without surprises, one where you don't have to do `console.log(Object.keys(thing()))` just to figure out what you can get back from `thing()`.

I like developing open-source modules because that forces me to document many more _"touch points"_ across my application stacks than I would have to otherwise. Let's face it, _we don't write documentation for every module, object, or class in our applications_. But, we are much more likely to write documentation for every module we open-source, at least for their public API. The more public API's we have, the better the documentation in your project becomes, **the easier it is to hunt down bugs**, and the easier it is to onboard newcomers into the project. Not to mention, open-source code usually follows most of the [12factor][2] design advice, which is always a great thing to do.

The more we rely on open-source, **the better our closed-source code becomes**.

> You could call it <mark>**API**-first</mark>, I guess.

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][1]._ Remember to **subscribe** if you got this far!

[1]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
[2]: http://12factor.net/ "The Twelve-Factor App"
