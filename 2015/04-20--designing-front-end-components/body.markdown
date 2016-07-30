After coming up with [Rome][1], I've used it as a template for other _"philosophically similar"_ front-end components, such as [insignia][2], [horsey][3], and [dragula][4]. Pretty much the only change I've made to the template in almost a year was switching from [Gulp][11] to [npm run][12] in the build process, and [that's just a matter of personal preference][13]. These libraries focus on doing one thing well, and share a theme across their API, demo page, and documentation. Furthermore, they've been designed to enable composability, or in other words, there shouldn't be any reason why one of these components couldn't be used with one or more other libraries.

For example, [Insignia attempts to improve the UX][17] on tag inputs by acting as a [WYSIWYG][5] input of sorts, where tags become easier to identify, remove, and manipulate by the user. It builds on the notion of [progressive enhancement][6], or the principle that **the web should mostly work** _"before JavaScript"_. To that end, existing input on a field is converted into tags, providing a consistent experience before and after JavaScript kicks in. Here is where I'd like to emphasize that sometimes we build things with UX in mind that are actually detrimental to the human experiencing what we've built.

> Always try and ask yourself, is this actually better for the end user or is it **just more productive for me** as a developer to do it this way?
>
> A lot of the time, the former simply isn't the case.

Composability isn't just a matter of [playing well with the web platform][7]. In practice, playing well with other libraries is just as important. Getting composability right is quite hard, how are you supposed to provide support for every other library out there that may interact with things you are using in the most unexpected ways imaginable? The answer is that you're not. You are not supposed, for example, to account for libraries that arbitrarily remove DOM nodes, move them, or wrap them in some other nodes. You are, however, supposed to **make sure you keep the noise your module produces to a minimum**, ideally producing zero noise.

There's a number of ways you can keep the noise to a minimum. One of them involves keeping CSS in check.

# Turn down the CSS

If you look closely at [any][1] [of][2] [the][3] [four][4] libraries I've mentioned thus far, you'll notice how simplistic their styling is.

[![An screenshot of Horsey's autocompletion feature in action][8]][9]

This is _decidedly intentional_, and aimed at making it easier to integrate the components with whatever styles your application may have. There's a few rules I follow with regards to styling within components.

- Shy away from inline styles
- Always prefix your CSS classes
- Keep styles to a minimum
- Distribute plain CSS along with the source

These seemingly simple rules have a deeper meaning, which I'll lay out next.

### Shy away from inline styles

Avoiding inline styles is crucial. Inline styles are really hard to override, leading to a lot of unnecessary `!important` rule-sprinkling. Most of the time, a CSS class will do a better job at grouping a set of styles together, making it easier for the developer to identify what the styles are trying to accomplish, change the component stylistically, and generally have more control over the styles of the component. Not every inline style can be avoided. Positioning styles, such as `left` `top` `right` and `bottom`, that frequently change over time have a place inline. Coincidentally, the developer is expected not to have a need for modifying these styles, where control is deferred to the library author to do as they intended.

### Always prefix your CSS classes

This one should be fairly obvious. Now that we've moved most of the styles in our library to CSS classes, we'll abide by this rule to create a semantic namespace for our component's classes. If you want a deeper reasoning into the topic I suggest you read [CSS: The Good Parts][10], which discusses best practices when it comes to writing and maintaining modular CSS.

This point seems almost trivial, why make such a big deal out of adding `rd-` at the beginning of every class name? What is a 2 letter prefix on every single class name going to do for me? It's probably not going to do as much for you, as a module author, since every class name will probably have the same prefix. However, when you start taking into account a few modules working in tandem, or an entire array of components working under the same application _(which has plenty of styles on its own)_,  creating faux namespaces via prefixing every class in a component becomes a glaringly obvious thing to do.

Namespaces make it faster to figure out what component a class name belongs to, help reduce class name clashes _(`.dialog` anyone?)_. This not only mitigates the chance that you mess up the existing styles of an application, but it also prevents the styles of that application from unwittingly breaking those of the component.

CSS is a two way street, but with the advent of flying cars we have to be extra careful.

### Keep styles to a minimum

Allowing the consumer some control over the styles in a component, and making sure those styles don't clash with others, will only have an impact as long as your component doesn't think too fancily of itself. The more styles you apply to a class name, the harder it'll be for a consumer to style that to their needs. Seriously, think about this. If you are providing me with a tag editing library, **why would you include styles for round borders and a box shadow on the input?** If the answer is purely stylistic _(it looks great!)_, then chances are those styles never belonged with the library in the first place.

Overstyling components is an easy mistake to make. After all, who wouldn't want to have this **sweet sweet transition** when the tag input moves after a tag is edited? It just takes a line of CSS! `transition: all 0.2s ease-in-out`. People will surely love that. It's fine to get fancy, but always strive to make it so that if a consumer doesn't want an style, he can get rid of it with a single line of CSS. If your users need to reset a lot of CSS in order to get a component to look and feel how they need it to, **that's on you**.

In fewer words, there shouldn't be anything stopping the consumer from **styling your component so that it perfectly blends into their designs**.

### Distribute plain CSS along with the source

Packaging the CSS in a component has always been a debated topic. Some argue that it should be bundled with the JavaScript, so that consumers get everything by simply including your module. Complicated solutions exist too, such as [Webpack allowing you to `require` CSS][14], whatever that may mean. My preferred approach is to compile CSS in every build from whatever source code I have, _which has been Stylus for a long time now_. I can then distribute that CSS, along with the source `*.styl` file(s), directly on GitHub. Nowadays my builds create a tag on `git`, and publish updates to both Bower and npm.

As far as consuming these modules goes, I'll typically install them from `npm`, include them with Browserify, and import any CSS into my Stylus bundle. Meanwhile, other users could just take the `*.css` files directly, whatever floats their boat. If you go through [articles on this blog tagged [build]][15], I've covered the topic pretty extensively so I won't go into further detail here.

I asked people on Twitter what specific concerns they had regarding modular front-end component design. One of the questions I got was about how to deal with CSS, which we've just covered. The other questions were concerned with [clean, well-abstracted API design][16], as well as effectively structuring modular code. These two go hand-in-hand.

<blockquote class="twitter-tweet" lang="es"><p>Writing a new article for <a href="https://twitter.com/ponyfoo">@ponyfoo</a> on designing modular front-end components. Is there a specific topic you&#39;d like me to address?</p>&mdash; Nicolas Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/589842423351746560">abril 19, 2015</a></blockquote>

# Be the API you wish to see in the world

Even though it may not seem that way, whenever I set out to build my own component I start by scouring the web for existing, usable alternatives to writing my own. Typically this yields a mixture of things I like and things I don't like, and helps shape the ideas I have about designing an API of my own. If you sift through a few libraries that sort of do what you want, you should be able to come up with a decent set of needs and use cases other people have, and add those to your own. Once you have that, the next step involves coming up with an API that's the most common denominator for all of those use cases.

> In the simplest possible use case, there should be **little or no configuration** involved.

When it comes to components for the front-end, I think the recipe that best fits most use cases is having an API consisting of a single function that takes a DOM element, and an entirely optional configuration object. You should be able to come up with a reasonable default behavior for your component that will just work by indicating the DOM element to use. Beyond that, a configuration object will provide the consumer with customization for the appearance of the component, as well as assist them in covering the more advanced use cases.

I think [Rome][1] is a great case study of this approach.

[![A screenshot of Rome in action][18]][19]

[Rome][1] just takes an HTML `<input>`, and the consumer is ready to go. Clicking on the provided element will pop a simple calendar right below the input, and the user can choose a date from the calendar. This is the most basic use case and shouldn't involve any configuration whatsoever. Therefore, it doesn't. The consumer may definitely have more advanced use cases than this. They may want to hide the time, as only the date matters to them. This is, again, a very common thing to wish for, so Rome makes sure that's easy to do: `rome(input, { time: false })`.

Some use cases are clearly more complicated than others, and they don't deserve a pinpointed solution such as a configuration flag, but rather a more generic solution. The example below changes the calendar so that only a certain date range is deemed valid and selectable. While an arguably advanced use case, having the ability to determine a valid date range is sure to up often enough in a date picker that being able to handle these directly as configurable options makes sense.

```js
rome(input, { min: '2013-12-30', max: '2014-10-01' });
```

The most important aspect of API design is to **find the right abstractions**, though. Suppose a library user creates an issue because they want to allow users to choose dates in summer, spring, or autumn, but not winter. You might be thinking "well, I could just create a configuration option called `seasons` with an array of the valid seasons. That sounds like a perfectly acceptable solution. It meets the user's needs, and it's abstracted enough to work for any season.

You've got to ask yourself, though. **Would you use such an option? Would you find it simple?** Most importantly, what happens when the next user comes along asking for a way to make the second week of each month invalid? Sundays? An specific point in time? Will you add a configuration option for each of these use cases? Of course not. Will you just cater to the needs of those who better align with the needs you have today for the library? Or will you go for a more generalized approach that just determined if any given date is valid?

In the case of Rome, the answer was to create a `dateValidator` option that allowed consumers to determine whether any given date was valid.
 
```js
rome(input, {
  dateValidator: function (d) {
    var m = moment(d);
    var y = m.year();
    var f = 'MM-DD';
    var start = moment('12-21', f).year(y).startOf('day');
    var end = moment('03-19', f).year(y).endOf('day');
    return m.isBefore(start) && m.isAfter(end);
  }
});
```

Yes, `min` and `max` are effectively shadowed by this option, and internally they could very well function as an alias for `dateValidator`. That's perfectly okay. Easy of use of an API endpoint should roughly match how frequently use cases come up in a given area.

In the same vein, you should strive to reutilize parts of the API effectively. Originally, Rome didn't have a way to _"link"_ two calendars, a common feature enabling the user to choose a _"start date"_ and an _"end date"_ for some sort of event or query. Given that I already had the generic `dateValidator` configuration property in place I could [come up with a few validators][20] that would determine if dates are valid for a given calendar based on the selection in some other calendar. Effectively building ranges, but without changing the public API, besides exposing the validators themselves.

```js
rome(left, {
  dateValidator: rome.val.beforeEq(right)
});

rome(right, {
  dateValidator: rome.val.afterEq(left)
});
```

Rome even goes as far as to provide the consumer with the ability [to rename all CSS classes][21].

# The API behind the API

Once we get past the API used to create a calendar using Rome, we get back an object containing an API of its own. This object can be used to interact with the calendar after it has been created. When following this pattern I like providing, as an starting point, a `destroy` method which removes any DOM elements and unbinds any event listeners produced when instantiating the component. This makes sure you play well with single page applications, highly dynamic views, and other advanced use cases. The other method I find quite necessary to include is a `.find` method on the public API that returns these instance API objects. Typically you'd do something like `rome.find(input)` and get back the calendar API associated with that `input`, or `null` if there wasn't any calendar associated to it yet.

Beyond these two methods, the API behind the API should be mostly in charge of exposing parts of the component you'll sometimes want to trigger manually. In the case of Rome, besides displaying the calendar when clicking or focusing on the input, the consumer may want to add a keyboard shortcut to display the calendar, and the API should be perfectly fine supporting that use case. Beyond these simple methods, the most crucial aspect of the API behind the API is events. Emitting a few events at important milestones, such as when a calendar is displayed or a date is selected, allows the consumer to have fine grained hooks and control over what the user does, and to manipulate the outcome of those actions every step of the way. These events could be handled through synthetic events on the DOM element itself, or using a library to convert your API into an event emitter. You can use [crossvent][22] and [contra.emitter][23] respectively for these two approaches.

I tend to keep the API behind the API really simple beyond that. Following a declarative configuration style is typically going to work wonders for your component, as there will be far less coding involved for the library consumer.

# `wrapping('up')`

As far as keeping your code modular effectively, you should treat every `function` in your code the same way as we discussed for the public API itself. Strive to make it simple, to identify common patterns and use cases and abstract away just enough so that you can accomodate for the most frequent use cases in the most elegant way, and then handle the least frequent use cases a bit differently. You should try and put together methods that make logical sense. Make an effort to give `function`s names that are as descriptive as possible _(but not too verbose)_, and keep everything that doesn't fit the name you gave a `function` in some other `function`.

Eventually, you should start to notice how the very methods you want to promote to public API citizenship map quite logically to what you already have. For example, when I was developing `dragula` I already had `cancel`, `remove`, and `end` methods by the time I wanted to promote them to _"API behind the API"_ level, which made it a very natural decision to make for me.

Lastly, I'd mention that keeping the dependency count low will make your modules more attractive to the public eye, as they'll become more decoupled from everything else and easier to integrate into whatever your potential consumers are doing.

[1]: https://github.com/bevacqua/rome "bevacqua/rome on GitHub"
[2]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
[3]: https://github.com/bevacqua/horsey "bevacqua/horsey on GitHub"
[4]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
[5]: http://en.wikipedia.org/wiki/WYSIWYG "WYSIWYG defined on Wikipedia"
[6]: /articles/tagged/progressive-enhancement "Articles tagged 'progressive-enhancement' on Pony Foo"
[7]: /articles/building-high-quality-front-end-modules "Building High Quality Front-End Modules on Pony Foo"
[8]: https://camo.githubusercontent.com/ba466d12f9a3175daa526c67b2cdf7f0e628df81/687474703a2f2f692e696d6775722e636f6d2f696d44464330432e706e67
[9]: http://bevacqua.github.io/horsey/ "Try a demo of Horsey online!"
[10]: /articles/css-the-good-parts "CSS: The Good Parts on Pony Foo"
[11]: /articles/my-first-gulp-adventure "My First Gulp Adventure on Pony Foo"
[12]: http://substack.net/task_automation_with_npm_run "Task automation with npm run, by @substack"
[13]: /articles/gulp-grunt-whatever "Gulp, Grunt, Whatever on Pony Foo"
[14]: http://webpack.github.io/docs/stylesheets.html "Stylesheets in Webpack"
[15]: /articles/tagged/build "Articles tagged 'build' on Pony Foo"
[16]: /articles/building-high-quality-front-end-modules "Building High Quality Front-End Modules on Pony Foo"
[17]: /articles/baking-modularity-tag-editing "Baking Modularity into Tag Editing"
[18]: https://cloud.githubusercontent.com/assets/934293/3803583/387125ea-1c1c-11e4-974e-467984e4d1f0.png
[19]: http://bevacqua.github.io/rome/ "Try a demo of Rome online!"
[20]: https://github.com/bevacqua/rome/blob/master/src/validators.js "Rome date validators on GitHub"
[21]: https://github.com/bevacqua/rome#default-options "Default options for Rome on GitHub"
[22]: https://github.com/bevacqua/crossvent "bevacqua/crossvent on GitHub"
[23]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
