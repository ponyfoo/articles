I made the clarification above because in other sorts of software, enterprise grade stuff, you might want to be more _"strict"_ about releases. Think <mark>Node.js or io.js</mark>. For the majority of us, though, **this is not the case**.

The **consensual definition of [semver][2]** more or less lies in the following bullet points.

- `patch` whenever you release bug fixes that don't affect the public API in the slightest
- `minor` whenever you make a change that can potentially affect the public API, regardless of it being a bug fix or a new feature
- `major` whenever you make a change that breaks expectations in the public API, such as removing an API touchpoint, changing its inputs, or its outputs

I prefer to take **a pragmatic approach** to versioning, as the above is usually a hassle. Bug fixes rarely break consumer code in practice, but almost always fall in the `minor` category. Consider a bug fix that someone else was using as a crutch and expected it to be a _"feature"_. That's a `minor` because it breaks an expectation, even though it was a measly bug fix, and even though it wasn't documented, and even though _(chances are)_ nobody _actually relied_ on it.

My approach is to consider these bug fixes that may or may not affect functionality a `patch`, rather than a `minor`. This brings me to point number two in this <mark>thought</mark>.

# Getting Rid of Hats

Hats _(and tildes)_ in `package.json` must die. Die in a fire. Die an ugly death where you burn them so they don't come back as undead white walkers to frostbite you into an ugly death. Becoming one of them, in turn, and devouring your loved ones.

I'm of course talking about `^` and `~`, but also about similarly dangerous operators like `>=` and `*`. Why would you take something as precious as `npm`'s immutability and throw it under the bus is beyond me -- yet it's the default configuration value for `npm`.

After all the hard work that was put into _(and is being put into)_ `npm` to fix dependency hell, using the mutable dependency operators ask too much from the community. Not everyone is going to be as gentle as you might think. I have had many dependencies break on my face after doing something innocuous like `rm -rf node_modules ; npm i`, and this shouldn't be something to worry about. Your ability to deploy safely shouldn't be lingering _just_ on tests that make sure your dependencies didn't break overnight.

There is **no good reason why dependencies should be able to break overnight**. I'll take my safe deployments over your _"minor bug fixes with chances of wreaking havoc"_ forecasts any day!

> _Run the following command. This <mark>**madness ends today!**</mark>_

```shell
npm set save-exact true
```

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][1]._ Remember to **subscribe** if you got this far!

[1]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
[2]: http://semver.org/ "SemVer has a specification"
