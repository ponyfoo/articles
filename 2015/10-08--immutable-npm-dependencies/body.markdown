## `npm` as a Platform

I concede getting bug fixes through quickly is great, but it might be best to leave those things to platforms such as [greenkeeper.io][1] to notify us of updates, so that we can implement them and bump packages up the dependency chain.

When it comes to the "but you'd have to wait for a fix to `dep-c` until `dep-b` bumps `dep-c`" case, I'd argue that **`dep-b` is already working with `dep-c`**, so why change it? At least `dep-b` wouldn't be **mutable**. If we ignored version ranges, all packages would be immutable and installing them would always yield the same bits of code.

Yes, semantic versioning ranges help with the version flattening hell but it's not exactly perfect either, you still end up with several different versions of programs even when you support ranges _(less of them, but still)_. If we ignored version ranges, we should find better ways to flatten the dependency tree. The way I see it, this is only a problem when using `browserify` with large applications, but not so much in the server-side.

In contrast, when `dep-c` or even `dep-d` introduce a bug, we'd have to fix `dep-b` and update our dependency on it, or we'd be screwed. This means that effectively our package is as safe as the most recent published packages. If we ignored version ranges, we'd be in charge of how `dep-b` works, and **`dep-b` would always be installed in the exact same way.**

## Security

This would also fix the case where a package down the dependency chain "goes dark" and is silently included into already published, trusted, popular packages. Sure everyone trusts `express`, but what if one of their dependencies deep down the chain suddenly decided to `publish` a version with malicious code in it? It would immediately creep into the user's application.

This isn't an easy problem to fix, but if we ignored version ranges, these security flaws wouldn't be automatically included into future installs of modules that were published **before the flaw was introduced**.

## Shrinkwrap

I don't think [`npm shrinkwrap`][2] is the solution to all of these problems. First, I've never seen a CLI app in the wild that's shrink-wrapped. That means CLI apps are prone to bugs caused by unexpected dependency updates. To use `shrinkwrap` when distributing CLI apps over `npm` would mean that you also have to decouple those CLI modules from any public API that they expose, as according to the semver logic, you wouldn't want to pin your API dependencies. Of course, decoupling CLI from API is a great idea, but also one only a handful of module authors follow.

Second, even with `shrinkwrap` at the application level, suppose somebody develops a perfectly working `dep-b` that depends on `dep-c`, and publishes it. If `dep-c` introduces an unexpected change that would break `dep-b`, those who installed `dep-b` before-hand and have a shrinkwrap handy will be protected. Those who install `dep-b` after `dep-c` introduced its breaking change will have a broken `dep-b`. The tests passed on `dep-b` at publish time, and there's no reason _(well, there shouldn't be a reason)_ for you to be writing unit tests that ensure `dep-b` works as expected. In this sense, semver is comparable to an infectious disease. You didn't have it before, and all was well, but now that you have it you're infecting everyone who comes near you.

Third, and really a reiteration of the first point, not a lot of people use `shrinkwrap`. If nobody uses `shrinkwrap`, does it really fix anything? Is the user to blame or should we  come up with a better solution that doesn't place the burden on the user?

## Should we Ditch Version Ranges?

A proposal to ignore semver ranges starting in the next major `npm` release might not be as crazy as it sounds. When a range is encountered, the oldest available version in the range could be used instead of the latest. This would effectively _"turn off"_ version ranges and make `npm` package installations more predictable. Predictability would prevent fringe, hard-to-trace bugs. We can still get fixes by collaborating, using tools like [greenkeeper.io][1], and finding better solutions for semi-automatic dependency chain updates than semver-style _"humans are trustworthy!"_ -- maybe something like the ["real-time break detection"][4] feature in greenkeeper.

<blockquote class="twitter-tweet" lang="en"><p lang="en" dir="ltr">Best fix to unpredictable/mutable npm installs&#10;- Ignore semver ranges in npm@latest&#10;- greenkeeper.io all the modules&#10;<a href="https://t.co/dtDgAbKtJm">https://t.co/dtDgAbKtJm</a></p>&mdash; Nicolas Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/652208794097815552">October 8, 2015</a></blockquote>

I understand that that wouldn't prevent mutability in dependencies that point to git repos, but those aren't as frequent in the wild and are most often found in application-level `package.json`. I don't have any hard data for that last bit, other than my experience with seeing very few `git+https` dependencies in `package.json` manifests in the wild.

The biggest problem with this approach is that we'd be losing the benefits of semver, and we're already deeply invested in semver as a community, so this is **probably not going to happen** and we'll have to embrace that mutable dependencies are pretty much a fact.

Here are some _actionable_ recommendations about what we can do in practice, instead.

## Immutable Dependencies with Shrinkwrap

If you are distributing a CLI application and you don't want your users to get unexpected patches that might break the CLI at any time, it's best to pin versions using a `npm shrinkwrap` step and then verifying the CLI works as expected before publishing. The same goes for applications. You should `shrinkwrap` and test applications before deployments to ensure everything is working properly before going to production. That way, you avoid unpleasant surprises down the road.

### Using `npm shrinkwrap` for Distribution

The easiest way to pin dependencies with shrinkwrap is adding a `prepublish` step to your `package.json`. This will ensure that before publishing your application, your dependencies are pinned.

```js
{
  "scripts": {
    "prepublish": "npm shrinkwrap"
  }
}
```

You probably want to omit `npm-shrinkwrap.json` from your git repository, as shrinkwrap is mostly meant to be _a release device_.

```shell
echo "npm-shrinkwrap.json" >> .gitignore
```

Note that you should `shrinkwrap` your modules only if you're dealing with an top-level application -- be it web, a CLI, or something else that's not meant to be consumed by other modules_. If we're talking about a CLI, that means you probably want to split the CLI itself from the API that powers it, and shrinkwrap only the CLI. The reason for that is that its API might be intended to be consumed as a library by other modules.  
If anything, keeping your CLI separated from the API that powers it is a good practice and you should be doing that anyways!

> Thanks to [Jordan Harband][3] for reviewing drafts of this article.

[1]: http://greenkeeper.io "Your software, up to date, all the time."
[2]: https://docs.npmjs.com/cli/shrinkwrap "Shrinkwrap API documentation"
[3]: https://twitter.com/ljharb "@ljharb on Twitter"
[4]: https://medium.com/greenkeeper-blog/announcing-real-time-dependency-break-detection-for-greenkeeper-4f7558c10d77 "Announcing “Real Time Dependency Break Detection” for Greenkeeper"
