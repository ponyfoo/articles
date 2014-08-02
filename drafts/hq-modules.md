# Building High-Quality Modules

Lately I've been developing front-end modules solely based on Browserify, the latest being [rome][1]. Rome is a calendar component that has an extensive feature-set. I've compiled a list of highlights below.

- Date _and time_ picker
- _Fancy_ [demo site][4]
- Use it on its own, or dedicated to an input
- Concise API and sensible defaults
- Date range just means validating two calendars against each other
- [Doesn't rely on jQuery][2]
- Works on **IE7+**

Rome wasn't conceived with all of these features out-of-the-box, though. When I released it for the first time, it was _just a native date-time picker_ with a reasonable API, but it complete lacked IE support, date validation, and the ability to be used without an input field.

[![colosseum.png][3]][4]

_Rome wasn't built in a day._

  [1]: https://github.com/bevacqua/rome "bevacqua/rome on GitHub"
  [2]: /2013/07/09/getting-over-jquery "Getting Over jQuery"
  [3]: http://i.imgur.com/jU8JmSs.jpg
  [4]: http://bevacqua.github.io/rome "Rome on GitHub Pages"

I believe that **one of the core drivers for high-quality modules is open-source**. Open-source _forces_ you to [think hard about the API][1] you're going to provide for your components, as well as put yourself in the shoes of an API consumer, and learn what you would like the API to be. Another module quality booster can be found in documentation. I love to thoroughly document the modules I build. This doesn't merely help outsiders, which would be too self-less. It also helps you pour your thoughts into words that explain the behavior of your API. If it's hard to describe, then chances are your API is hard to use as well.

[Automating your build process][2] is another crucial piece of the high-quality module puzzle. An [automated build **(and release)** process][3] allows you to easily deploy new releases to [`npm`][4], [`Bower`][5], as well as creating a git tag on your public repository. Maintaining [a changelog][6] is almost a requirement when you are trying to keep your consumers up to date regarding your latest changes. Changes to your API should be reflected in both the documentation as well as the changelog, while internal code quality refactorings don't necessarily need to be reflected in the documentation.

Remember: documentation should be relevant to the consumer, not the module author. This means that implementation details shouldn't be included in the documentation, because the consumer is only concerned about the public-facing API.

# Staying Away From jQuery

I developed Rome out of frustration, because I couldn't find a single date picker that had a reasonable API, didn't depend on jQuery, and still had good browser support. Every single component I found had something I disliked, maybe they didn't allow the human to change the date by typing into the input, they had a `<select>` for the hour, another one for the minutes, and another for the period _(AM or PM)_. If features weren't the issue, their API was wildly incoherent, like the ones you find in jQuery UI components, or the component depended on jQuery.

Don't get me wrong, [jQuery][7] is great if you're into it, but you must understand that [it isn't the be-all and end-all of JavaScript][8]. When it comes to components, abusing jQuery is even worse, because every dependency you decide on means that it's another dependency you are burdening the consumers with. This, in turn, signifies that the consumer will have to consider whether your library is worth adding that dependency for. Maybe you've developed [a great JavaScript grid component][9], but deciding on jQuery meant that anyone who doesn't want jQuery in their project is unable to leverage your library. This issue is intensified if we consider libraries that depend on jQuery UI just for its dragging or resizing capabilities. I've also seen things that depended on jQuery UI just because it adds animation and CSS color functionality into the core jQuery feature-set. That's just plain wrong, inadmissible.

> As module authors, we should be more thoughtful with regards to the consumer.

What if someone wants to use your component with Angular? Angular already comes with [a "lite" version][10] of jQuery, why would they use jQuery as well? What if they are already using one of the myriad of other jQuery-like libraries out there? Are they supposed to use two of those now, because your stubborn component expects them to use just-such bloated library? I don't think so. A better approach is to learn about the native DOM API across different browsers, and use that to your advantage. I'm not saying you should re-implement jQuery from scratch, but at the very least do your consumers a favor and use [a custom build][11], or go for a module that just [does what you actually need][12] out of jQuery.

Now that that rant is out of the way, let's talk API.

# API Design is Tough

The best way to produce a high-quality API is to start with that. Ask yourself what API you would like to use.

> Ask yourself what an attractive API should behave like

The API isn't only about what methods to expose, but also about being consistent throughout all of those methods, and making it terribly simple to use. A great API is intuitive, meaning that the consumer is able to _correctly_ guess what a method is called, or how an option is going to be named, or how the component is going to react to different configuration. Designing an API to be intuitive takes using lots of different APIs and learning over time what makes them tick.

In the case of Rome I started with a single `index.html` file with some CSS and JS in it that contained what the API should look like.

```html
<input id='foo' />
```

```js
rome(foo);
```

Making that available on a `github.io` page was just a matter of creating a `gh-pages` branch and pushing to it.

```shell
git checkout -b gh-pages
git push -u origin gh-pages
```

A nice screenshot always helps drive people to the demo page. This is a great testing ground for both consumers and ourselves, when testing out new pieces of functionality.

[![gh-pages.png][13]][14]

The great thing of having a live demo page readily available is that people will help you with blatant bugs, even if they're only mildly interested on your project.

Once you've bolted down a basic API, you can start defining what the default functionality should look like. In this case I decided choosing both a date and a time would be reasonable defaults, while being able to turn either off setting `date: false` or `time: false` in the _(also optional)_ options object that you can provide to the component.

  [1]: /2014/01/20/how-to-design-great-programs "How to Design Great Programs"
  [2]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
  [3]: /2014/01/27/my-first-gulp-adventure "My First Gulp Adventure"
  [4]: http://npmjs.org/ "npmjs.org"
  [5]: http://bower.io/ "bower.io"
  [6]: https://github.com/bevacqua/rome/blob/master/CHANGELOG.md "CHANGELOG.md for bevacqua/rome on GitHub"
  [7]: http://jquery.com/
  [8]: /2013/06/10/uncovering-the-native-dom-api "Unconvering the native DOM API"
  [9]: https://github.com/tonytomov/jqGrid "jqGrid on GitHub"
  [10]: https://github.com/angular/angular.js/blob/master/src/jqLite.js "jqLite source for angular/angular.js on GitHub"
  [11]: http://projects.jga.me/jquery-builder/ "jQuery Builder"
  [12]: http://sizzlejs.com/ "Sizzle.js JavaScript Selector Engine"
  [13]: http://i.imgur.com/sGBQPsG.png
  [14]: http://bevacqua.github.io/rome "Rome on GitHub Pages"

[rome open-source javascript jquery gulp modularity]
