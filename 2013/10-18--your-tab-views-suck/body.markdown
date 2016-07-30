## Why would I even want that? I have jQuery UI for that!

![jquery-ui.png][1]

Sure you do, and that's why you want these. You **don't need** jQuery. Consider an example taken straight from the [jQuery UI Tabs](http://jqueryui.com/tabs/ "jQuery UI Tabs Documentation") page. First, let's glance at the HTML.

```html
<div id="tabs">
  <ul>
    <li><a href="#tabs-1">Nunc tincidunt</a></li>
    <li><a href="#tabs-2">Proin dolor</a></li>
    <li><a href="#tabs-3">Aenean lacinia</a></li>
  </ul>
  <div id="tabs-1">
    <p>some lipsum</p>
  </div>
  <div id="tabs-2">
    <p>some lipsum</p>
  </div>
  <div id="tabs-3">
    <p>some lipsum</p>
  </div>
</div>
```

That is exactly what we want, non-repetitive HTML that just marks up the tabs and their content. This is as good as it gets, though, because then there's the JavaScript.

```html
<script>
  $(function() {
    $( "#tabs" ).tabs();
  });
</script>
```

Okay, okay, we're fine with this as well, sort of. It's _just a selector_, and a function. You should be wondering why the heck we'd need to wait [until DOM ready](https://developer.mozilla.org/en-US/docs/Web/Reference/Events/DOMContentLoaded "DOMContentLoaded event explained on MDN") to cash in some tabs, though. Your _real problem_ should be with having to pile up 7 extra HTTP requests just so you can get a few tabs there, and _then_ wait until those finish so that you can have some tabs show up.

```html
<link rel="stylesheet" href="http://code.jquery.com/ui/1.10.3/themes/smoothness/jquery-ui.css" />
<script src="http://code.jquery.com/jquery-1.9.1.js"></script>
<script src="http://code.jquery.com/ui/1.10.3/jquery-ui.js"></script>
```

Ever wonder what goes on in the [Chrome DevTools](https://developers.google.com/chrome-developer-tools/ "Chrome Developer Tools") _network tab_? **Mayhem**, pure mayhem.

![network.png][2]

So it's not even just those 3 resources, which by the way amount to well **over 200k**, _compressed_. Apparently we're fetching four small images as well, just because we can. This time, compressed to... **oops!** _two times their original size_.

Never mind the _well known rule_ to [always push scripts to the bottom](http://developer.yahoo.com/performance/rules.html#js_bottom "Yahoo Performance Rules, put #js on the bottom"), which you can't follow now, because otherwise your precious tabs would [FOUC (Flash of Unstyled Content)](http://www.paulirish.com/2009/avoiding-the-fouc-v3/ "Avoiding the FOUC by Paul Irish") all over your human, which we'd never want to happen. jQuery recognizes that last one, so they silently put all of their example code in the `<head>`, and now it's _your problem_.

> **7** HTTP requests, **200k** in our pocket, and a **DOMContentLoaded** event later...

At long last! We're now able to render these _awesome looking_ UI tabs! Yes!! Except [they look _really_ hideous](http://jqueryui.com/resources/demos/tabs/default.html "jQuery Tabs UI Example"), and with a bunch of classes, like `ui-tabs-nav ui-helper-reset ui-helper-clearfix ui-widget-header ui-corner-all`, they are _virtually impossible to style_.

![jquery-tabs.png][3]

Luckily, we don't need **jQuery** _for any of this_. In fact, I'd argue that **jQuery UI** is utterly useless, if it weren't for drag and drop, and particularly the [sortable](http://jqueryui.com/sortable/ "jQuery UI Sortable") functionality. That being said, that's a piece of **jQuery UI** people _rarely need_. You most certainly don't need that kind of thing to create a tabbed view. So you have a combination of rarely used components mixed and matched with completely useless ones. True, you _can_ download a customized package that trims away the fat components you don't need, but did you even bother? That won't fix the _through-the-roof HTTP request count_ issue, anyways. Let's forget about **jQuery** for a while, _try to think_, and resolve this without using it.

## Of course, there's a better way

Okay, so no jQuery. We've got a clean slate. Good! How do we get tabs? _Hmmm..._ Let's start with what we know, like all ambitious projects do. The HTML was _mostly fine_, we just ditched the JavaScript. What is the single best way to toggle state without using JavaScript at all? The answer lies in `<input>` types `radio` and `checkbox`. The latter isn't all that useful to us, we need to be able to toggle between tabs, not turn them on and off. Only one should be enabled at any given point, this is _a perfect scenario_ to use radio buttons!

Radio buttons have **state**, and only one can be checked at any time. Let's imagine _a world without tabs_. Something like the code below would be enough for us, then.

```html
<input type='radio' name='tab-group' class='tv-radio' checked />
<input type='radio' name='tab-group' class='tv-radio' />
<input type='radio' name='tab-group' class='tv-radio' />
```

That's great for **state**, but we'd like some content for each tab to show up with that. Well, consider adding a `<div>` after each of those inputs.

```html
<input type='radio' name='tab-group' class='tv-radio' checked />
<div class='tv-content'>Tab 1</div>
<input type='radio' name='tab-group' class='tv-radio' />
<div class='tv-content'>Tab 2</div>
<input type='radio' name='tab-group' class='tv-radio' />
<div class='tv-content'>Tab 3</div>
```

Using a combination of the [`:checked`](https://developer.mozilla.org/en-US/docs/Web/CSS/:checked "CSS :checked pseudo-selector on MDN") pseudo-selector and the [`+` _next sibling_](http://css-tricks.com/child-and-sibling-selectors/ "Child and Sibling Selectors explained on CSS-tricks") selector, we'd quickly be able to show one of those `<div>`s, depending on _which radio button_ is on! _Amusing stuff_.

```css
.tv-content {
  display: none; /* hide tabs next to unselected radios */
}

.tv-radio:checked + .tv-content {
  display: block;
}
```

This is fine and minimalistic, but we can agree [it looks even worse](http://codepen.io/bevacqua/full/jIkvf "Example on CodePen") than the jQuery UI version. It's not even close to usable, and styling radios is not doable.

![radios.png][4]

## Half-way there now...

The radios are ugly, and they don't belong, thus, we need to hide them. However, that presents the fundamental inconvenience that we can't click on them now. _Or can we?_ Time to pull another trick from the bag, this time we'll turn to `<label>` elements. Labels allow us to use the `for` attribute, making it so that when we click on them, the related input gets the click, too. This reference requires an `id` attribute on the inputs, but that's no big deal.

The HTML ends up looking like below.

```html
<label for='tab-1'>Tab 1</label>
<label for='tab-2'>Tab 2</label>
<label for='tab-3'>Tab 3</label>
<input type='radio' name='tab-group' class='tv-radio' id='tab-1' checked />
<div class='tv-content'>Contents 1</div>
<input type='radio' name='tab-group' class='tv-radio' id='tab-2' />
<div class='tv-content'>Contents 2</div>
<input type='radio' name='tab-group' class='tv-radio' id='tab-3' />
<div class='tv-content'>Contents 3</div>
```

The only tweak to the CSS is hiding the radio buttons.

```css
.tv-radio {
  display: none;
}
```

Still _not really cute_, but easy to style, and [barely uses any code](http://codepen.io/bevacqua/pen/BIjvH "This time with labels! Batteries not included. On CodePen").

![labels.png][5]

Below is a screenshot of how they ended up looking _after a tad of styling_.

![tabs.png][6]

The source code is up on [GitHub](https://github.com/bevacqua/untab "Tabbed UI View on GitHub") and [CodePen](http://codepen.io/bevacqua/full/qxnDw "Tabbed UI View on Code Pen").

## `<wrapping>up</wrapping>`

We've spent a good chunk of this article looking at the problems with assuming libraries _just know better_, and how we might be _perfectly able to home-bake a cake and eat it, too_.

The **jQuery** library excels at reducing conflicts across the board (even though you [don't really need it](/2013/07/09/getting-over-jquery "Getting Over jQuery") as badly as you might think), but their _UI framework_ does little about that, and a lot about drastically increasing your application's _network bandwidth usage_.

What other solutions did you learn about where you didn't really need _any JavaScript_ at all? I'd love to hear!

  [1]: https://i.imgur.com/VsEVdRk.jpg "jQuery UI: Everything you've never wanted"
  [2]: https://i.imgur.com/Wh0wOPT.png "Those are some very necessary requests!"
  [3]: https://i.imgur.com/FqwdjEM.png "All that for a measly 200k?"
  [4]: https://i.imgur.com/CGOzi1t.png "Not quite what we envisioned..."
  [5]: https://i.imgur.com/AeqV7mH.png "No more circles!"
  [6]: https://i.imgur.com/rUBqxhh.png "Flat, clean, and gorgeous!"
