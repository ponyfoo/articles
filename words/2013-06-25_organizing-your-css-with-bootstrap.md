# Organizing your CSS with Bootstrap #

Now that we've laid [the basics](/2013/06/24/css-for-dummies "CSS For Dummies") in the cascading land of awesomeness that is CSS, it's time to move forward and take a deeper look at _organization and tooling_.

Any decently-sized codebase, even CSS, needs some sort of organization in order to keep us from going insane. Pre-processing, and a few ground rules for style writing, will help us achieve the _level of organization **required** in higher level applications_.

## Organization ##

Before we jump onto template scaffolding with bootstrap, I wanted to go over some tips to improve the organization in the way we write our CSS code. Take these as general rules, apply them _as you deem necessary_.

- Use classes for styling. Whenever possible, _avoid using element IDs_, attribute values, and tag names
- Try not to nest style rules _too deeply_. To prevent complexity, limit yourself to _one or two levels_ of nesting
- Avoid _magic pseudo-selectors_ such as `:nth-child` or `:nth-of-type`. Use an extra class in your markup, instead
- Don't overqualify selectors. i.e: `div.foo.bar`, when you could use `.foo`
- Prefix classes with `js-` when referenced by JavaScript code
- Avoid styling `js-` classes
- Toy with the idea to use `box-sizing: border-box` [everywhere](http://www.paulirish.com/2012/box-sizing-border-box-ftw/ "* { box-sizing: border-box; } FTW")
- Attempt separating layout styling concerns from design concerns. Keep _separate stylesheets_

These are just a few ground rules, feel free to add your own or _adapt_ these to your own needs. You might want to use [SMACSS](http://smacss.com/ "SMACSS Style Guide") as your _guiding light_.

### Lint your CSS! ###

If you have a large enough project, CSS can get out of hand as easily as JavaScript, or even more so, because developers seldom pay attention to CSS. You are [already using JSHint](/2013/03/22/managing-code-quality-in-nodejs "Managing Code Quality in NodeJS"), which is great. If you really want to take your codebase to the next level, you should _at least consider_ linting the CSS, too.

Currently, you can use [CSSLint](https://github.com/stubbornella/csslint "CSSLint on GitHub") as your lint tool. If you are using LESS, you are in luck! [RECESS](https://github.com/twitter/recess "Twitter RECESS on GitHub") will let you lint your LESS code directly.

**Ideally**, you should include a _lint task_ for CSS in your [build process](/2013/05/22/understanding-build-processes "Understanding Build Processes").

## Pre-processors ##

Pre-processors allow us to keep our stylesheets [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't Repeat Yourself Principle"). They also let us perform calculations in our layout measurements, or do color shifting.

Quite probably though, the most useful feature of CSS pre-processors is the ability to use mixins, reusable functions that allow us to write cross-browser styles without having to type them time and again, risking typos and duplicating our code.

The most popular flavors of CSS pre-processing probably are:

- [SASS](http://sass-lang.com/ "Syntactically Awesome Stylesheets"), from the universe of Ruby and [CoffeeScript](http://coffeescript.org/ "CoffeScript language")
- [Stylus](http://learnboost.github.io/stylus/ "Stylus pre-processor"), from the awesome [LearnBoost](https://github.com/LearnBoost "LearnBoost on GitHub")
- [LESS](http://lesscss.org/ "LESS CSS language"), still rocking it

Lets do a quick comparison of the three:

|              | SASS         | Stylus       | LESS         |
|-------------:|:-------------|:-------------|:-------------|
| Syntax       | CSS-like     | Customizable | CSS-like     |
| Compiler     | Ruby gem     | Node package | JavaScript   |
| Verbosity    | Higher       | Lowest       | Regular      |
| Source       | [GitHub](https://github.com/nex3/sass "SASS on GitHub") | [GitHub](https://github.com/learnboost/stylus "Stylus on GitHub") | [GitHub](https://github.com/cloudhead/less.js "LESS on GitHub") |
| Framework    | [Compass](http://compass-style.org/ "Compass CSS Authoring Framework") | [Fluidity](https://github.com/InkSpeck/fluidity "Fluidity Framework") | [Bootstrap](twitter.github.com/bootstrap/ "Twitter Bootstrap") |

All three pre-processors share similar syntax and features when it comes to color functions, mixins, and variables. You can find a more detailed, _feature by feature_ comparison [here](http://net.tutsplus.com/tutorials/html-css-techniques/sass-vs-less-vs-stylus-a-preprocessor-shootout/ "SASS vs LESS vs Stylus").

I should mention _all three_ are readily available as grunt tasks on npm.

> In my opinion, it depends on what you want to do. If you are going to use a full-fledged CSS framework, I'd go with LESS, because Bootstrap is _a clear winner_ in the field.

> If you are going to use the framework as is, or not going to use one, you could probably do with Stylus. It's clean looking syntax is [really appealing](https://gist.github.com/paulmillr/2005644 "Gist showing simplicity of Stylus").

SASS used to be regarded as _a superior language_ than LESS. Today, they are _very similar_. SASS feels a _little more verbose_, though. That, coupled with the fact that **Bootstrap** is powered by LESS, makes me choose _LESS over SASS_ with little hesitation.

## Twitter Bootstrap ##

![bootstrap.png][1]

[Bootstrap](http://twitter.github.io/bootstrap/ "Twitter Bootstrap Framework") is a CSS framework, which _encompasses_ a lot of the practices we mentioned earlier. It does:

### Rapid Prototyping ###

We've all been there before. Trying to get a design right, but instead, we _wasted our time_ dealing with `float` issues, with margins, paddings, and footers that wouldn't stick to the bottom of our page.

Bootstrap provides an [scaffolding module](http://twitter.github.io/bootstrap/scaffolding.html "Bootstrap Scaffolding") that allows us to very quickly set up the basics for our layout. The scaffolding module provides us with a grid system that allows us to quickly place elements on our designs with ease.

At this point, you'll be probably better off [downloading Bootstrap](http://twitter.github.io/bootstrap/assets/bootstrap.zip "Download bootstrap.zip") and playing around with it for a while. But I'll try my best to give you a reasonable example.

This grid system uses 12 columns. These columns have a fixed width, and allow you to quickly throw together a multi-column layout:

```html
<link href="/css/bootstrap.css" rel="stylesheet">

<div class="row">
    <div class="span3">Menu</div>
    <div class="span9">Content!</div>
</div>
```

[Pen here](http://cdpn.io/mtgsj "Basic Layout Example").

You can also nest these columns, and it will still work, just make sure to use _a new row element_:

```html
<div class="row">
    <div class="span3">Menu</div>
    <div class="span9">
        <h1>Articles</h1>
        <div class="row">
            <div class="span6">Article 1</div>
            <div class="span6">Article 2</div>
        </div>
        <div class="row">
            <div class="span6">Article 3</div>
            <div class="span6">Article 4</div>
        </div>
    </div>
</div>
```

[Pen here](http://cdpn.io/yegbn "Nested Layout Example").

As you can see, laying out your design is _almost trivial now_. All we need to do, is fill up the template with _actual content_. Keep in mind we should _always wrap our templates_ in a container like this:

```html
<div class="container"></div>
```

### Responsive Layouts ###

![devices.png][2]

Bootstrap also offers [fluid layouts](http://twitter.github.io/bootstrap/scaffolding.html#fluidGridSystem "Fluid Grid System") which, rather than _having a fixed width_ for each column, assign each column _a percentage_ of the viewport real estate instead.

Adding an [additional stylesheet](http://twitter.github.io/bootstrap/scaffolding.html#responsive "Enabling responsive features"), you'll enable responsive design features.

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="/css/bootstrap-responsive.css" rel="stylesheet">
```

When you include the responsive stylesheet, styles will be added so that when the viewport size is adjusted, the layout adjusts itself to it, providing _a more flexible user experience_ that works better across multiple platforms.

This is achieved using [media queries](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries "CSS media queries"). Here are some media query examples:

```css
@media (max-width: 600px) {
    /* styles enabled when the viewport is at most 600px wide */
}

@media (min-width: 700px) and (orientation: landscape) {
    /* styles enabled when the viewport is at least 700px wide
       AND in landscape orientation, too
     */
}
```

Media queries allow CSS to be applied **exclusively** if the media matches the query, at any given time. Usually, a few media query breakpoints are carefully picked. This allows you to _plan your layout_ for two or three of the expected media devices used when visiting your application.

[Here](http://css-tricks.com/snippets/css/media-queries-for-standard-devices/ "Media queries for standard devices") is a pretty comprehensive list of media queries.

I will surely come back to the subject in later articles, but for now, I'll recommend you two really good books about responsive design.

#### [Responsive Web Design](http://www.amazon.com/dp/098444257X "Responsive Web Design by Ethan Marcotte") ####

This book covers _responsive web design in depth_, showing you lots of examples. It is a very good resource to introduce you to the fantastic world of responsive design for today's web. A **must read** if you're _even faintly interested_ in RWD.

#### [Mobile First](http://www.amazon.com/dp/1937557022 "Mobile First by Luke Wroblewski") ####

Closely related, Mobile First introduces you to a new way of thinking about design. Rather than trying to cram a critical mass of ads in your mobile designs, Luke suggests to start with the small screens instead. Only presenting the essential content first, and building from there. Adding content as we hit breakpoints, instead of removing it.

This may seem like a very subtle difference from traditional web development, and it is. But it induces a new way of thinking about what's important to the user, and it helps you cut features and content you didn't really want there, nor even need.

### More Bootstrapping ###

Its features don't just end with scaffolding. A comprehensive list of [basic styles](http://twitter.github.io/bootstrap/base-css.html "Fundamental HTML Styles") for tables, forms, buttons, and images, is also included. Right out the box!

These are _pretty straightforward_, so I won't go into detail, how hard can it be to _create a large button_?

```html
<button class="btn btn-large btn-primary"></button>
```

That's _it_.

Bootstrap also offers an [icon set](http://twitter.github.io/bootstrap/base-css.html#icons "Glyphicons in Bootstrap") you could use _without any further customization_.

### Components and JavaScript ###

Twitter also offers a decent amount of [components](http://twitter.github.io/bootstrap/components.html "Reusable components in Bootstrap"), and widgets that require just [a little of JavaScript](http://twitter.github.io/bootstrap/javascript.html "JavaScript in Bootstrap") to get them going.

You should _make sure to check those out_ before embarking yourself in a **component-creating fiesta** and not even realizing someone else _already did all the work for you_.

  [1]: http://i.imgur.com/TTMpDxW.png "Twitter Bootstrap CSS Framework"
  [2]: http://i.imgur.com/3hTIoim.png "Different Media Devices"