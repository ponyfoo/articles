# The beauty of TAP

TAP being <mark>a precisely defined format</mark> means that any programs that can process or output TAP can be connected through UNIX pipes. Like most good things in programming, this is deceitfully simple. Consider, for example, the case of reporters. There are [over a dozen TAP reporters][2] listed on the `tape`. This is a familiar concept in test tooling. The tool generates output that can then be formatted, _in turn_, by a reporter that highlights the types of information we care about _(failing tests, diffs, nyan cats, etc)_. TAP can also be used for transformations, such as the hacks code coverage tools like [coverify][3] do to figure out how good your code coverage is.

Then there's continuous testing flows and continuous integration. Let's start with the former.

# Continuous Testing with `hihat`

[`hihat`][5] is a shiny new package created only two months ago, but it's potential and usability are both huge. Let me just demonstrate using a screenshot.

[![A gif depicting hihat in action][4]][5]

You may have noticed that in the demo they use `tap-dev-tool`. This is a reporter that prettifies TAP for our particular use case -- rendering the tests on a DevTools instance. The way `hihat` works is really cool. It uses [Electron][7] behind the curtains, the runtime used under [GitHub's Atom editor][6]. In case it's not obvious from the demo, `hihat` reloads the DevTools whenever you make a change to your code.

If you're anything like me, your first impression when you saw that screenshot was **"that's too good to be true"**, and your second impression probably was something along the lines of _"there's no way that's easy to set up"_.

Turns out, it's freaking easy to set up. First, install things.

```shell
npm i -D hihat tap-dev-tool
```

<sub>_In case you were wondering, `i` is one of plenty of shortcuts for install, and `-D` aliases `--save-dev`._</sub>

Then, make a slight change to your `package.json`.

```json
{
  "scripts": {
    "test": "hihat test/*.js -p tap-dev-tool"
  }
}
```

Now you can just run the tests that you used to run simply in Node, **within a DevTools instance** _(effectively, in Chromium)_, and it will watch for changes! Super amazeballs.

# Continuous Integration with `testron`

A very similar and complementary tool to `hihat` is [`testron`][8]. Testron takes the output from your `browserify` build and runs it through [`electron`][7] as well. Testron is designed to run on the command-line, so you won't get any fancy DevTools. But, Testron does make it dead simple to run your browser tests on Chromium.

Just like `hihat`, `testron` is dead simple to use.

```shell
browserify test/* | testron
```

That's it!

[![Testron running our client-side tests on Travis][11]][12]

# Bonus: `errorify`

During development, you can save precious think-shark-tank seconds by using [`errorify`][9]. It's meant to be used together with [`watchify`][10], and it'll `tee` compile errors into the terminal and a bundle whose sole purpose is to tell you that the bundle failed to compile from within the HTML.

This way, you can save some time figuring out _"oh, it failed to compile, probably missed a semicolon"_, or similar situations. Stop tabbing between your terminal and your browser today! 

[2]: https://github.com/substack/tape#pretty-reporters "Pretty TAP reporters"
[3]: https://github.com/substack/coverify "substack/coverify on GitHub"
[4]: https://camo.githubusercontent.com/c6f8071b302eb0bfd62fe50d97599b4071e52ad5/687474703a2f2f692e696d6775722e636f6d2f537170626a7a6c2e676966
[5]: https://github.com/Jam3/hihat "Jam3/hihat on GitHub"
[6]: http://atom.io/ "Atom Editor"
[7]: https://github.com/atom/electron "atom/electron on GitHub"
[8]: https://github.com/shama/testron "shama/testron on GitHub"
[9]: https://github.com/zertosh/errorify "zertosh/errorify on GitHub"
[10]: https://github.com/substack/watchify "substack/watchify on GitHub"
[11]: https://i.imgur.com/1lhObnf.png
[12]: https://travis-ci.org/bevacqua/dragula "bevacqua/dragula on Travis"
