# CSS For Dummies #

Web design today is hard to get right. I've been meaning to talk about front-end design for a while, but I couldn't get the subject _quite right_. Seeing how detailing the [underlying browser technology](/2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API") in JavaScript, I figured I'd do the same for [CSS](https://en.wikipedia.org/wiki/Cascading_Style_Sheets "Cascading Style Sheets").

I'll be taking a different approach, though. Rather than explain what libraries do, I'll try explaining why a need for them was born, and walk you through the most basic stuff, such as selectors, and follow up by tackling complex topics such as fonts, [Bootstrap](http://twitter.github.io/bootstrap/ "Twitter Bootstrap CSS Framework"), pre-processing, and more.

## Fundamentals ##

CSS was born out of necessity. The necessity to _separate content from presentation_. The idea was to put behind stuff like `<body bgcolor='black'>`, and work towards a more layered approach. DOM elements would get CSS classes (more on that later), and in our CSS, we would define style rules, such as `background-color`, or `font-size`.

The previous example would be redefined as:

```html
<head>
    <style>
    body {
      background-color: black;
    }
    </style>
</head>

<body>
```

The CSS is contained in a `<style>` block, or it can be alternatively be placed in an external stylesheet. Like so:

```css
/* style.css */

body {
  background-color: black;
}
```

```html
<head>
    <link rel='stylesheet' href='style.css'>
</head>
    
<body>
```
    
Alternatively, you can set styles in DOM attributes, or through JavaScript. The former is hardly ever recommended, with an _exception_ being made in the case of _HTML emails_. The latter is sometimes useful when _calculations are needed_, but it's generally bad practice otherwise. JavaScript is useful for toggling (adding and removing) CSS classes on DOM elements, to denote a change in state.

```html
<body style='background-color: black;'>
```

```js
document.body.backgroundColor = 'black';
```

Note how the notation changes in JS. As a general rule, style properties are [camel-cased](http://en.wikipedia.org/wiki/CamelCase "Camel Casing - Wikipedia") in JavaScript, and hyphenated in CSS.

#### Classes ####

We mentioned _classes_ earlier. Classes can be styled in CSS style sheets by prefixing them with a dot.

```css
.my-class {
  background-color: black;
}
```

They can be added to DOM elements using the `class` attribute. _Multiple classes_ can be added to each element.

```html
<body class='my-class'>

<body class='my-class that-one another'>
```

#### The Cascading Aspect ####

Thus far, we've been discussing _Style Sheets_, but remember, these are also **Cascading**, hence CSS. So what does _cascading_ imply?

Cascading means that **order matters**. That is to say, styles follow a [convoluted](http://www.w3.org/TR/CSS2/cascade.html "Cascading and Inheritance in CSS - W3") order of precedence rule-set. This order of precedence can be overridden appending `!important` to the end of our style values, but _this is a hack_, and **not recommended**.

Instead, we should design our style sheets so that our rules cascade into each other. If we find ourselves resetting a bunch of properties from a _less specific selector_, in a more specific one, that should signal trouble (more on selectors in a minute).

#### Selectors ####

Selectors are what we use in CSS to identify the elements we want to style with a particular set of rules. If you are a JavaScript developer, you probably already used selectors with jQuery, or with the [native methods](/2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API") that underline its selector engine.

A quick overview of selectors:

```css
/* elements with the given id */
#element-id /* matches <anything id='element-id' /> */

/* elements with the given class name */
.class-name /* matches <anything class='class-name' /> */

/* span DOM elements */
span /* matches <span/> */

/* elements with an href attribute */
[href] /* matches <anything href /> */

/* data-foo property with value equal to 'bar' */
[data-foo=bar] /* matches <anything data-foo='bar' /> */

/* 'star selector', matches everything */
* /* matches <anything /> */

/* the currently focused element */
:focus /* matches <anything />, when active */

/* elements that are under your mouse pointer */
:hover /* matches <anything />, when you mouse over it */

/* selectors can be chained using spaces or special characters */

/* strong tag, one of its parents is an span */
span strong

/* strong tag, its immediate parent is an span */
span > strong

/* span tag immediately following a .title element */
.title + span 
```

You can go deeper into selectors looking at the [W3 recommendation](http://www.w3.org/TR/CSS21/selector.html "CSS 2.1 Selectors - W3"), but these should be more than enough to get you going.

- Keep your selectors as simple as possible. This will make them easy to read and interpret later on
- You'll want to use _classes_ to style your elements
- Try and _avoid nesting styles too deeply_
- Following an [style guide](http://css-tricks.com/css-style-guides/ "CSS Style Guides - CSS-Tricks") might prove useful

#### CSS Properties ####

If you want to learn the different CSS properties you can apply, I'll recommend a few links.

- [CSS 2.1 properties](http://www.w3.org/TR/CSS21/propidx.html "CSS 2.1 Full Property Table - W3") in the W3 recommendation
- [css3files.com](http://www.css3files.com/ "CSS3 Files") explains what's new in CSS 3
- [css3please.com](http://css3please.com/ "CSS3 Please!") contains some cross-browser CSS rules

Honestly, _the best way to learn CSS from scratch_, other than the fundamentals I describe in this blog post, is **by googling**. You want to style something in a particular way? Just google it. In all likelyhood, someone _had_ that question before, and someone else _answered it_.

You can always learn more about CSS by subscribing to sites such as [css-tricks.com](http://css-tricks.com/ "CSS Tricks by Chris Coyier").

#### Cascading Issues ####

Earlier on I mentioned cascading should be _natural_. Lets look at an unnatural example.

```css
.button {
    border: 3px solid #ffc;
    border-radius: 5px;
    text-shadow: #999 0 1px;
    background-color: #fcc;
    padding: 8px;
    color: #f00;
}

.button.flat {
    border-radius: none;
    text-shadow: none;
}
```

Generally speaking, we'd be better off declaring this using _a more natural approach_.

```css
.button {
    border: 3px solid #ffc;
    background-color: #fcc;
    padding: 8px;
    color: #f00;
}

.button.rounded {
    border-radius: 5px;
    text-shadow: #999 0 1px;
}
```

If the reason for this isn't immediately apparent, try thinking about this in a broader scope, where you have _tons_ of overriding classes and many of them _don't want the 3D appearance_.

## History of CSS ##

Now that we know the basics about _what CSS is_, and how to apply it to _style the web_, lets dive into a **history lesson** for a while. Or, at the very least, a few milestones in CSS history, so that we better understand the current state of the web.

In the beginning, different kinds of browsers started emerging. [User agent stylesheets](http://www.iecss.com/ "UA Style Sheets"), shipped with each browser, presented different basic styles, and that represented a problem for web designers who wanted their site to look the same on every navigator. As a solution to this problem, the CSS reset concept was conceived.

#### CSS Reset ####

The main issue was with paddings, and margins. A first, naive fix for this problem, was to use the star selector, and simply reset padding and margin styles for every single element.

```css
* {
    margin: 0;
    padding: 0;
}
```

This, however, raised [_performance concerns_](http://www.stevesouders.com/blog/2009/03/10/performance-impact-of-css-selectors/ "Performance Impact of CSS Selectors"), and imposed unwanted resets on some elements, such as [lists](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Consistent_list_indentation "Consistent List Indentation - MDN").

Thus, a better approach was born, and eventually prevailed. Eric Meyer created the [CSS Reset](http://meyerweb.com/eric/tools/css/reset/ "CSS Reset Stylesheet"), which is an improvement over the former. It resets more than just margins and paddings. It avoids using the star selector, but instead _explicitly declares_ the tags it resets.

![reset.png][1]

In my mind, the main issue with `reset.css`, is the fact that it makes it really hard inspecting an element's styling, since really long reset statements are added on almost every single element, making it hard to read.

A more modern approach exists today.

#### Normalize.css ####

> A modern, HTML5-ready alternative to CSS resets

Rather than reset everything, [Normalize](http://necolas.github.io/normalize.css/ "Normalize.css alternative to resets") resets _specific styles_, in _specific elements_, and is [thoroughly documented](http://necolas.github.io/normalize.css/2.1.2/normalize.css "normalize.css source").

![normalize.png][2]

This has the _added advantage_ of avoiding the long lists of CSS reset styles we see in our HTML inspectors when resorting to a CSS reset. As you can see, normalize offers _a cleaner approach_.

#### Browser inconsistencies ####

As you might know from JavaScript, browsers don't always implement things the same way, or at the same time. This is _particularly nefarious_ when it comes to CSS 3 properties. As a result, some rules have to be written in several different ways if we want to maximize our [browser support](http://caniuse.com/ "Can I Use"). For example, we might need to write `border-radius`, to give a rounded border to an element, like this:

```css
-webkit-border-radius: 5px;
-moz-border-radius: 5px;
border-radius: 5px;
```

We should avoid doing this kind of repetitive, and _error-prone_ task by hand, and resort to a _pre-processor_ such as [SASS](http://sass-lang.com/ "SASS pre-processor"), or [LESS](http://lesscss.org/ "LESS pre-processor"), to help us keep our code DRY and concise. We'll go deeper into these topics in a later post.

#### CSS Grids ####

On another level, frameworks have been looming over CSS for a while now _(more on frameworks in a future post!)_, with different end goals.

There are quite a few different _grid systems_ such as [960.gs](http://960.gs/ "960 Grid System") and [semantic.gs](http://semantic.gs/ "Semantic Grid System"), to name a few.

Grid systems basically consist of a bundle of classes that help you rapidly prototype a website's layout using the classes provided by the framework. These classes help you define how many columns each element takes up in a grid's row. Here's an example taken from the [960.gs](http://960.gs/ "960 Grid System") site:

```html
<div class="container_12">
  <div class="grid_7 prefix_1">
      <div class="grid_2 alpha">
          ...
      </div>
      <div class="grid_3">
          ...
      </div>
      <div class="grid_2 omega">
          ...
      </div>
  </div>
  <div class="grid_3 suffix_1">
      ...
  </div>
</div>
```

Before you jump on the grid system bandwagon, I'd advise you to check out [Twitter Bootstrap](http://twitter.github.io/bootstrap/ "Twitter Bootstrap"). Bootstrap is a _comprehensive_ rapid prototyping solution for the front-end.

I'll write more on bootstrap in a future post, but I wanted to introduce you to the magic world of CSS first.

The next article regarding CSS will include topics such as:

- Intelligent ways to organize your CSS
- CSS pre-processor options available (what, and why _included_)
- A guide on what [Bootstrap](http://twitter.github.io/bootstrap/ "Twitter Bootstrap") is, how to _use it_, and why it rocks

Later on, I intend to write about more advanced topics

- Picking the right typeface for your site
- Responsive web design and mobile first
- Flat design!

If you're interested in these topics, make sure you _subscribe_ via [RSS feed](http://blog.ponyfoo.com/rss/latest.xml "RSS Feed for blog.ponyfoo.com") or through _email notifications_!

  [1]: http://i.imgur.com/IlF1a3X.png "reset.css in web dev tools"
  [2]: http://i.imgur.com/ePrmACd.png "normalize.css in web dev tools"