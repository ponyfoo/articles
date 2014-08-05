# Building High-Quality Front-End Modules

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

> Ask yourself what an attractive API should behave like, and _implement that!_

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

Now the demo is up at `{user-name}.github.io/{repo-name}`, which in this case is [`bevacqua.github.io/rome`][14]. A nice screenshot on the **README** always helps drive people to the demo page, as well as signaling some seriousness applies to the project. This is a great testing ground for both consumers and ourselves, when testing out new pieces of functionality.

[![gh-pages.png][13]][14]

The great thing of having a live demo page readily available is that fellow open-sorcerers will help you with blatant bugs, even if they're only mildly interested on your project. Better yet, you yourself will be able to spot bugs such as inconsistencies in your API, or irritatingly missing portions of functionality. For example, when I first put together `rome` it was barely a date-time picker that latched itself onto input fields. Then I realized people may want it decoupled from an input field, maybe for read-only purposes. Later, I added date ranges and validation that allows you to deem specific dates invalid, in those cases where you want to disallow holidays or just sundays, for example.

### _Flexible_, yet consistent

I consider flexibility to be **one of the qualities all great APIs seem to have**. Take a look at the [jQuery API][15]. It almost feels like you could throw anything at it and it'll just work, most often doing what you want it to. The key is to be flexible on the inputs, and consistent on the outputs. This means that when you're taking input from the API consumer, you should allow them to throw anything at you, within reason. In the case of `rome`, I allow input dates as native `Date` objects, [`moment`][16] objects, or date strings. Internally, I always use `moment` objects, because of their sheer power when it comes to computing date arithmetics, _something I didn't want to burden myself with_. Date manipulation is [incredibly hard to get right][17].

Being flexible is great for inputs, but in the case of outputs its **consistency** what you want. This means that if you allow chaining, then any method that doesn't have an explicit return value should return the API object instance itself. Whatever the return value for any given method, it should be properly documented so that consumers aren't surprised. You may want to blow their minds with what your library can do, but you definitely don't want to blow anyone's mind with an unstable API producing varying results depending on the provided inputs.

### _Flexible_, yet reasonable

Being flexible doesn't mean being insanely hard to configure. I think this is where most API designers get it wrong. It's not just about providing a bunch of options for every single action your library can take, but it's also about being able to determine sensible defaults that the consumer can accept most of the time. This way, if you do things right, people won't have to modify the default behavior unless they want to reach a more complex functionality quota.

Once you've bolted down a basic API, you can start defining what the default functionality should look like. In this case I decided choosing both a date and a time would be reasonable defaults, while being able to turn either off setting `date: false` or `time: false` in the _(also optional)_ options object that you can provide to the component.

Another sign of being reasonable is leaving room for composability. Face it, **your API shouldn't be able to do just about anything**. It should be able to do _just enough_, but just enough in ways that can be put together and interact with each other.

Consider as an example how you could create a link between two calendars, where one is a start date and the other is an end date. Here's how you would do it with `rome`. Just like I mentioned earlier about flexible inputs, these validators accept a DOM element associated with a calendar, a `Date`, a `moment`, or a date string.

```html
<input id='start' />
<input id='end' />
```

```js
rome(start, { dateValidator: rome.val.beforeEq(end) });
rome(end, { dateValidator: rome.val.afterEq(start) });
```

This is how jQuery achieves the same functionality.

```html
<input id='start' />
<input id='end' />
```

```js
$(function () {
  $('#start').datepicker({
    onClose: function (value) {
      $('#end').datepicker('option', 'minDate', value);
    }
  });
  $('#end').datepicker({
    onClose: function (value) {
      $('#start').datepicker('option', 'maxDate', value);
    }
  });
});
```

What if the calendar is inline? Surely there won't be any `'close'` events in that case, and you'd have to look for other examples to achieve the same functionality. Suppose that now you want to ignore some dates, with a raw approach like the `dateValidator` used in Rome, you would just have to return `false` for invalid dates. In the case of jQuery datepicker, the API leaks knowledge about the inner workings of the calendar, exposing a `beforeShowDay` method for which I'll just paste their API documentation here rather than _try to explain it_ myself.

> A function that takes a date as a parameter and must return an array with:
>
> `[0]`: `true` or `false` indicating whether or not this date is selectable  
> `[1]`: a CSS class name to add to the date's cell or `''` for the default presentation  
> `[2]`: an optional popup tooltip for this date

So, in order to determine that a date is invalid I'd have to do `return [false, '']`. **Eww!**

In contrast, having something like `dateValidator` which **the calendar just knows to call whenever it needs to validate a date**, you could centralize date validation and let the consumer blissfully ignore internal implementation details such as that you'll call that method whenever a day is rendered on the calendar to know whether it's a valid date for the human to pick. Nevertheless, this is probably _the least_ of **jQuery UI's API problems**, especially when pitted against jQuery, which has quite a pleasant API.

> The bottom line is that your API shouldn't be an assortment of corner-case-handling options, but rather provide a few concise solutions to abstract problems. Abstract enough that you can get away with defining a few of those and still allow consumers to use it for a variety of different use cases.

This brings me back to ditching jQuery, which leaves us with a grab-bag filled with browser quirks you'll need to take care of.

# Browsers Are Hard, But Worth Learning

Time and again jQuery has come up as an excuse for completely ignoring the [API provided natively][8] by different browsers, but the truth is that nothing is going to replace learning about what jQuery does under the hood. There's a few ways you could go about that. You could delve into the jQuery source, and familiarize yourself with what and why jQuery does things the way it does, [you could try to develop a cross-platform component or website][19] that doesn't rely on jQuery to work out the kinks across different browsers, or you could simply [read about those things online][18].

Some of the most common quirks you'll have to deal with along the way to becoming a cross-platform module-hurling ninjasaur legend, _— many of which I had to deal with when putting together `rome` and others before it —_ are listed below.

### Polyfills

Rome started as cutting-edge browser material, **breaking every rule about [progressive enhancement][21] out there**, but it eventually made its way into the dark heart of **IE7**. One of the most straightforward changes you have to make to support old browsers are [polyfills][20], which are drop-in pieces of code that can _"turn on"_ new functionality in older browsers that don't natively support it.

In the case of Rome, I required [quite a few of these][22] polyfills, as I happen to be quite fond of the functional array methods introduced in ES6. You don't always have it that easy.

Some times new functionality demands special behavior, such as implementing getters or setters, but most of the time you'll find that there are ways to work around these limitations even in the oldest of browsers.

> What you **can't polyfill**, You can _work around!_

### Working Around New Behavior

DOM events are a clear example of this case. In some versions of IE, DOM nodes don't even share a prototype, and in these cases it's just best to take the few cases where you were attaching event listeners through the [`addEventListener`][25] API and replace that with a cross-browser method that doesn't have to be a polyfill.

The difference lies in that using a polyfill shouldn't require you to change any of your existing code that currently functions on newer browser implementations. Meanwhile, by changing your existing code you could simply create a wrapper around the native browser APIs in a cross-browser fashion, just like jQuery does _— but at a smaller scale —_ **only for the desired browser API, and not for all the things**. In the case of Rome I implemented [my own cross-browser event handling][24] implementation and adjusted the places I used [`addEventListener`][25] to use that instead.

### Compromising On Known Limitations

(clone) https://github.com/bevacqua/rome/blob/master/src/clone.js

### `innerText` vs `textContent`

### The `'touchend'` event

This one is quite awkward, but as it turns out iOS isn't very fond of clicks on focused input fields, and as a result they won't fire any `'click'` events on a field that's already focused when tapped again. For my calendar component, this meant that tapping on an input field would only work before selecting a date for the first time, and if the focus didn't change you wouldn't be able to get the calendar to show up ever again. Typically, you can [get rid of this issue][23] by listening for the `'touchend'` event as well, which fires when the finger is lifted, as expected.

# Bare Minimum 

(styling, bottom line)

Open source is **such fun**, are you _willing_ to give it a try?

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
  [13]: https://cloud.githubusercontent.com/assets/934293/3803583/387125ea-1c1c-11e4-974e-467984e4d1f0.png
  [14]: http://bevacqua.github.io/rome "Rome on GitHub Pages"
  [15]: http://api.jquery.com/ "jQuery API Documentation"
  [16]: http://momentjs.com/ "moment.js "Parse, validate, manipulate, and display dates in javascript."
  [17]: http://stackoverflow.com/q/6841333/389745 "Why is subtracting these two times (in 1927) giving a strange result?"
  [18]: https://developer.mozilla.org/en-US/ "Mozilla Developer Network"
  [19]: https://github.com/bevacqua?tab=activity "My Open-Source Activity on GitHub"
  [20]: http://remysharp.com/2010/10/08/what-is-a-polyfill/ "What is a Polyfill?"
  [21]: http://en.wikipedia.org/wiki/Progressive_enhancement "Progressive Enhancement on Wikipedia"
  [22]: https://github.com/bevacqua/rome/tree/master/src/polyfills "Polyfills in bevacqua/rome@master on GitHub"
  [23]: https://github.com/bevacqua/rome/commit/fb8fc070fd4bc8b49009bff4b34c1b904f80a025 "'touchend on input to show' commit on GitHub"
  [24]: https://github.com/bevacqua/rome/blob/master/src/events.js "Cross-browser event handling in Rome"
  [25]: http://mdn.beonex.com/en/DOM/element.addEventListener.html "addEventListener on MDN"
  
[front-end open-source javascript jquery gulp modularity]
