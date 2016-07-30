# Trust Issues

A few hours ago, a discussion started on [`npm/npm#12045`][npmtrust] about whether we should trust `npm`. Trusting `npm` as a company is beside the point, though. A more important question is whether we can trust what people publish on `npm`.

> Can we trust package authors?

There are about `Infinity` possible security risks posed by an `npm` package author "going rogue", having his account compromised, or even making a mistake. Let's enumerate a few scenarios where if _"a package author decided to `${action}`"_ would result in pain and suffering for the ecosystem as a whole.

- Unpublish a popular module preventing their dependants, and their dependants' dependants from ever being installed again unless `npm` chimed in
- Include a `postinstall` script such as `rf -rf /`
- Include a `postinstall` script such as `npm unpublish`
- Include a `postinstall` script that allows for remote code execution
- Publish a semver patch version containing a bug that makes the package unusable

Even if we make `npm unpublish` harder *or even impossible*, the community's reliance on semver means that we would still be exposed to multiple other vulnerabilities, that may well fly under our radars, and that could be introduced at any point in time by a bad actor such as a rogue package author or someone taking over the account of a popular package holder.

The vast majority of `npm` users are benevolent, though. This is why semver mostly works.

> Trusting package authors mostly works. **Until it doesn't.**

Thus, we came up with techniques such as `npm shrinkwrap` or simply [bundling your dependencies on `npm publish`][publish] so that we can trust a snapshot of what package authors produce, and not all of it. How is doing that any different than getting rid of semver, something that has [been posited in the past without gaining much traction][semverimmul]?

Again, semver mostly works, so we're reluctant to not rely on it. There is value to semver, though.

# The Fix: Bundling Before Publishing

Bundling on publish means we take advantage of our dependencies being semantically versioned during development, but that we won't allow ourselves to publish a package that may mutate over time. At the same time, installs will be faster because we won't be relying on `npm` to resolve a bunch of dependencies. This is basically [what Rich argues in his Medium article][publish], and partly what I've argued [in my article about semver][semverimmul].

Does bundling on publish mean our dependants lose the ability to take advantage of semver?

> *No.*

# Semantic Versioning Can Still Help us During Development

Not allowing semantic versioning to leak through the entire dependency tree is the *healthy approach to take here*.

This is no different to Browserify having transforms run only at the local package level: the package author knows what transforms are best for their package, but they don't really know their dependencies, or whether something will break at some point by running those transforms on a global scale. The author of dependencies can't foresee every possible transform that'd be run against their code, so Browserify compromises on defaulting to local transforms. Semantic versioning is the same thing, but it's lacking a sensible default.

Then there's the issue of code duplication, but that's a story for another day. I'll take code duplication over fear of bringing services down any day. When it comes to front-end development, code duplication should be taken seriously, though, and `npm` is probably the best place where we can come up with a solution to that problem.

# Immutability, Predictability, and a Possibly-Service-Provided Solution

Another take on fixing this issue would be having an immutable version of `npm`, let's call it `ipm`. In this scenario, we'd keep semver for local development but there'd be a twist: anything ever published to `ipm` would remain on `ipm` forever. Unpublishing would only be possible through a DMCA process. At the time of publishing an `example@version` package, `ipm` would take care of computing all dependencies for `example@version`, respecting semver throughout the dependency graph, and bundling them together with the published `example@version` package. When `example@version` is installed, **the exact same content would be downloaded every time**. There wouldn't be any need for package authors to take on the obnoxious task of bundling things together, because the `ipm` service would take care of that for them. The service should also be smart enough to bundle packages in such a way that code duplication is reduced to a minimum.

A service like `ipm`, when widely adopted by the community, would accomplish two things.

**First**, situations such as a package author unpublishing a popular module and causing `ipm install` to fail across the board would be avoided by preventing user-powered unpublishing entirely. If a DMCA notice or similar legal reason were to trigger the take-down of the `doge` package from the registry, then things that depended on `doge` would remain unaffected because `doge` was previously bundled into their dependants. Similarly, if a malicious user decided to introduce a `postinstall` vulnerability, it would only affect development environments and those who decide to update their `doge` dependency without verifying its integrity through one of those monitoring solutions which ensure packages on `npm` don't go rogue.

> Trusting others is the foundation of our community, but everyone can make mistakes and some can go rogue. **We can't rely on trust alone.**

**Second**, and perhaps even most importantly, it'd introduce predictability, something we've been sorely missing lately in the grand scheme of all things web development. By staying immutable at the source, `ipm` would allow intermediary services such as Travis _-- as well as `npm` end users --_ to heavily cache those immutable packages that are known never to change. Even if development versions take full advantage of _(one level deep)_ semver ranges, each package downloaded with `ipm install` would be a never-changing bundle. Installation time would also significantly go down, even when the bundles aren't cached, `ipm` would have pre-computed all the necessary files for each dependency, no more tree-deep request-fests! 🌳

> Predictability shouldn't be as undervalued as it is today.

If anything, predictability speeds up our development and deployment processes. At its best, it prevents mistakes that stem from _"lazy developers not bundling their code before deployment for every single package they create ever"_. Of course, I disagree with the notion of lazy developers -- that's an oxymoron. But, as the lazy developers we strive to be, this problem should be resolved at the source, and not at the individual community contributor level. It's far too important an issue for each individual to be expected to take responsibility. The service needs to step up.

[npmtrust]: https://github.com/npm/npm/issues/12045#issuecomment-200976024 "'Should I trust npm?' - #12045 on GitHub"
[publish]: https://medium.com/@Rich_Harris/how-to-not-break-the-internet-with-this-one-weird-trick-e3e2d57fee28 "How to not break the internet with this one weird trick"
[semverimmul]: /articles/immutable-npm-dependencies "Keeping Your npm Dependencies Immutable on Pony Foo"
