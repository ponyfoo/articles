[![woofmark demo][1]][2]

The example shown above is [`woofmark`][3], and _it **underscores** just how useful composability can be_. I've developed `woofmark` for [Stompflow.com][4], as to build upon the Markdown editor that's available here on _Pony Foo_. <mark>Woofmark provides the _added benefit_ of being able to interpret HTML and WYSIWYG</mark> as well, which shouldn't be understated. Meanwhile, however, I'm able to use the Markdown parser on _Pony Foo_, even though it was originally built for Woofmark and [Stompflow][5]. Thanks to loose coupling, and since **it wasn't built into the editor**, I can get away with using it elsewhere.

If this was a monolithic framework for _Markdown, HTML, and WYSIWYG editing_, I would have, in contrast, a terribly large code base, much larger than that of `woofmark` _(which I already consider monolithic!)_, with a few added drawbacks.

- I wouldn't be able to use [`insane`][6] as a general-purpose lightweight HTML sanitizer
- I wouldn't be able to use [`megamark`][7] on the server-side to produce the same HTML output that the client-side displays
- I would have to implement something like [`crossvent`][8] everywhere I want to deal with DOM events, unless I'm willing to drop [jQuery][9] into everything I develop
- I wouldn't even know where to start poking at [`jsdom`][10], which is immensely huge in terms of a web browser, but only used server-side
- I wouldn't be able to use any of these modules across multiple code-bases, unless I did lots of copy-pasting and _ignored the benefits of having bugs fixed on a global scale_

Sharing code across multiple projects, or even just [Node.js][11] and the browser, is too big of a productivity boost to oversight. Yet, the most of us are still not buying into modular development because _"the asking price"_ is too high to get in the front door, but <mark>in reality we're missing out on being that much more productive in the long term</mark>.

# Tag and Drop

Consider the example below, where we use [dragula][12], _a drag and drop library_; and [insignia][13], a tag editing input enhancement, to provide a highly usable tag editing experience. Contrary to `woofmark` and many of the modules that surround it, **neither of these libraries where designed with each other in mind**. Composability was, however, considered when designing both of them. In the case of `insignia`, this simply meant allowing the consumer to start off with an `<input/>` element that may contain space-separated tags. Once JavaScript kicks in, `insignia` converts those tags into pretty DOM elements that can be styled and whatnot. This allows the `insignia`-consuming application to function even when `insignia` fails to load: _a minimal `<input/>` element with some tags in it will still be available_.

In the simplest of cases, `insignia` converts the `value` to tags and takes over user input by binding a flurry of keyboard event listeners on `input`. Note how simplistic the API is at this level.

```js
insignia(input);
```

Of course, you can always rack up complexity, _with both [`dragula`][14] and [`insignia`][15]_, setting many different `options` and customizing what can be done with them. Keeping down the complexity and progressively increasing the difficulty with which a consumer can customize your component _may prove hard_, but it's also [the best way to deliver an experience][16] that <mark>can be digested over time</mark>. They start out using your **simplest use case**, and then they may _discover more advanced use cases_ as they go over your documentation or keep using the component. This way you keep the barrier of entry low while the usability _(and applicability)_ of your module stays high. Notable examples of this sort of architecture include [`uglify-js`][17], [`tape`][18], [`browserify`][19], and [`dragula`][20] _(if I may say so myself)_, among others.

[Dragula][21] is an entirely different animal than [`insignia`][22]. It's goal is to <del>feed on the blood of others</del> provide a thin interface between humans and the underworld of drag and drop. It doesn't make lots of assumptions about what your use case is, so **it can remain flexible**. <mark>The primary assumption [`dragula`][23] makes is that you probably want to be able to drag things between one or more _containers_</mark>. These containers, `dragula` asserts, will have any quantity of top-level children waiting to be dragged away and dropped somewhere else. Or in _the same_ container, providing the ability to re-order elements within a container. This effectively covers most use cases, there's many other things you could do with `dragula`, but for the main use case, it _feels too good to be `true`_:

```js
dragula(<mark>[container1, container2, container3]</mark>);
```

Now you're able to drag any top-level children of `container1`, `container2`, and `container3`, and drop them back onto any of those containers.

[![Screenshot of demo page for Dragula on GitHub Pages][24]][37]

[Insignia][25] has a constraint, it demands that the consumer places their `<input/>` as the single child of another DOM element. That `<input/>` then gets `<span>` siblings on both sides, where the tags are placed. The reason for this constraint is that it translates into a benefit that, on its own, **justifies choosing [`insignia`][26] over any other tag editing library out there**: the ability to seamlessly move between tags simply by using the arrow keys, without flickering or stuttering.

When the user wants to navigate to any given tag, every tag between the `<input/>` and that tag is moved out of the way. Consider the following example, where the user clicked on the <mark>highlighted</mark> `[tag]`.

```html
[tag] <mark>[tag]</mark> [tag] <input/> [tag] [tag]
```

In this case, every tag to the right of `<mark>[tag]</mark>` that's not already on the right of the input is moved to the right. Then the highlighted `[tag]` gets removed, and the input assumes it's value. This sounds highly invasive, but in reality it's exactly the opposite. Focus never leaves the `<input/>`, and thus is never lost, which is really important when trying to improve the rudimentary _(yet reasonable, and often underestimated)_ UX of entering tags by typing some text into an `<input/>` field.

Besides meaning that the UX provided by [`insignia`][27] is actually worthwhile,<sup>*</sup> , the way in which it operates is quite unobtrusive with regards to the DOM, as it doesn't do much more than move _(or add)_ elements between the two siblings to the `<input/>`. Before going to a demo and showing you how these two libraries work together, look at this piece of code which is all the JavaScript used to tie both pieces of the puzzle together.

```js
var input = document.querySelector('.input');
var result = document.querySelector('.result');
var tags = insignia(input);
var drake = dragula(<mark>{
  delay: true,
  direction: 'horizontal',
  containers: Array.prototype.slice.call(document.querySelectorAll('.nsg-tags'))
}</mark>);

input.addEventListener('insignia-evaluated', changed);
drake.on('shadow', changed);
drake.on('dragend', changed);

function changed () {
  result.innerText = result.textContent = tags.value();
}
```

The highlighted options in [`dragula`][28] are needed because:

- `delay` allows click events to get through before being considered drag events
- `direction` isn't required, but it makes it smoother for `dragula` to figure out where tags should be dropped
- `containers` is just both of the tag containers created by [`insignia`][29], casted to a true array

Whenever a new tag is evaluated by `insignia`, or a drag event ends in `dragula`, the result gets refreshed. Refreshing the result whenever `dragula`'s shadow moves isn't all that necessary, but it does provide _an interesting boost to perceived performance_!

<p data-height="360" data-theme-id="9622" data-slug-hash="oXPgaV" data-default-tab="result" data-user="bevacqua" class='codepen'>See the Pen <a href='http://codepen.io/bevacqua/pen/oXPgaV/'>Composable UI: Insignia and Dragula</a> by Nicolas Bevacqua (<a href='http://codepen.io/bevacqua'>@bevacqua</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

# Tag Completely

Remember how I bragged about how unobtrusive [`insignia`][30] is? It's not just useful for doing things with the elements around it, but you can also get away with relying on the `<input/>` itself not doing anything funky too.

In this example, we'll mix [`insignia`][31] with [`horsey`][32], a general-purpose autocomplete library that also doubles as a drop-down list _(and why not, a **"combo-box"** too, whatever that may be)_. Horsey can be used to add autocompletion features to an `<input/>`, a `<textarea>`, or even to non-input elements like a `<div>`, _effectively becoming a drop-down list_. Autocompletion is added via a list that can be controlled using the keyboard, just like [`insignia`][33], or by clicking on the suggestions.

Rendering the list of suggestions has a default implementation that just takes a string, but you can also use any templating engine you want to render the list items. In the screenshot below, the "fruits" were rendered by adding an image tag along with the text.

[![Suggestions rendered with a fruit by their side][34]][35]

The code barely even changes, we're still creating a tag editor with `insignia(input)`, but we're now using `horsey` to add **an autocompletion feature**. Of course, there were some styling changes, but those _aren't as interesting_.

```js
var input = document.querySelector('.input');
var result = document.querySelector('.result');
var tags = insignia(input);

<mark>horsey(input, {
  suggestions: [
    'here', 'are', 'some', 'tags',
    'and', 'extra', 'suggestions',
    'ponyfoo', 'dragula', 'love',
    'oss'
  ]
});</mark>

input.addEventListener('insignia-evaluated', changed);
<mark>input.addEventListener('horsey-selected', changed);</mark>

function changed () {
  result.innerText = result.textContent = tags.value();
}
```

Here you get suggestions on what tags to enter next, and you can also amend a previously entered tag just by opening the autocomplete list and picking a different tag, pretty neat! Again, all of this is possible because [`horsey`][32] doesn't take any radical actions on the `<input/>`, it just helps you pick a value and places it's suggestions below the input, but that's it! There's no further DOM alteration coming from [`horsey`][32], which is just what [`insignia`][33] needs.

<p data-height="340" data-theme-id="9622" data-slug-hash="RPYxKE" data-default-tab="result" data-user="bevacqua" class='codepen'>See the Pen <a href='http://codepen.io/bevacqua/pen/RPYxKE/'>Composable UI: Insignia and Horsey</a> by Nicolas Bevacqua (<a href='http://codepen.io/bevacqua'>@bevacqua</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

# A Horse that <mark>Barks</mark>

Horsey even works in `<textarea/>` elements, following the caret **(text cursor)** around and whatnot. This makes it the ideal companion to [`woofmark`][36], if you have entities you want to hint at: issue references, _like <mark>#40</mark>_; at-mentions, _like <mark>@bevacqua</mark>_; or _anything else_.

While [`woofmark`][36] is based on legacy code and hence quite abysmal to look at, it does a good job of keeping large chunks of code in other modules. It's up to you to provide a Markdown to HTML parser, as well as an HTML to Markdown parser. Of course, you get recommendations. You should probably use [`megamark`][7] as your Markdown parser, and [`domador`][38] as the DOM parser.

The code below is what's used in the demo to tie [`woofmark`][36] and [`horsey`][32] together. In the first highlighted block you'll notice that we're using tthe pure _"distro"_ versions of both [`megamark`][7] and [`domador`][38], although in practice you'll probably want to _wrap them_ in your own methods and **customize their behavior**. Since we're using the distros, **we'll just turn off fencing**, the ability to parse triple <mark>**\`\`\`**</mark> backticks back and forth. Otherwise, we would have to add some more code to detect the programming language when parsing HTML back into Markdown. _Not something we want to do for a simple demo._

```js
var textarea = document.querySelector('.textarea');
var editor = woofmark(textarea, <mark>{
  parseMarkdown: megamark,
  parseHTML: domador,
  fencing: false
}</mark>);

horsey(textarea, {
  suggestions: [
    '@bevacqua', '@ponyfoo',
    '@buildfirst', '@stompflow',
    '@dragula', '@woofmark', '@horsey'
  ],
  anchor: <mark>'@'</mark>,
  editor: <mark>editor</mark>,
  getSelection: <mark>woofmark.getSelection</mark>
});
```

We had already played a bit around with [`horsey`][32], so what are all the new highlighted options? While everything we've seen so far is composed, I've cheated a little for `woofmark`, and so `horsey` helps you out if you want to use it with `woofmark`. The reason for this is that I usually have them working side-by-side in my projects. In hindsight, `horsey` shouldn't take a Woofmark `editor` instance, because that's very tightly coupled. Instead, an intermediary module should bridge the gap. It'd still be reusable, but `horsey` itself wouldn't need to know about woofmark anymore.

Woofmark has the ability to switch between user input on a `<textarea>` for Markdown and HTML, or a `<div contentEditable>` for WYSIWYG editing. In this sort of <mark>long-form user input</mark>, it makes the most sense to _append the suggestion_, provided by `horsey`, onto what you already have on the input. The default behavior for `horsey`, _which makes the most sense on inputs_, is to **replace** the value altogether. The `anchor` property is used to determine _when_ the suggestions should pop up. In this case, as soon as we see a `@` character. When a suggestion is chosen, anything before the suggestion that matches it will be "eaten". Suppose I've typed `@beva` and pressed <kbd>Enter</kbd> on the suggestion to enter `@bevacqua`, Horsey will figure out that `@beva` was the value to autocomplete, and it'll just add `cqua` to that.

The reason why I've inserted knowledge about `woofmark` in `horsey`, originally, was that I still needed some of the same logic to deal with `<textarea>` elements on `horsey` anyways. In hindsight, _again_, I should've just come up with a higher level abstraction that could be reused via a third module. Similarly, there's nothing stopping `woofmark.getSelection` from being its own standalone module, as [that's just a polyfill][39].

> _There's always room for improvement!_

I'll go get my modularity affairs in order. In the meanwhile, check out the <mark>Woofmark + Horsey</mark> demo [on CodePen][40]!

<p data-height="525" data-theme-id="9622" data-slug-hash="bdxajE" data-default-tab="result" data-user="bevacqua" class='codepen'>See the Pen <a href='http://codepen.io/bevacqua/pen/bdxajE/'>Composable UI: Woofmark and Horsey</a> by Nicolas Bevacqua (<a href='http://codepen.io/bevacqua'>@bevacqua</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

<mark>**P.S**</mark> How obnoxious do you think the _highlights_ are? I probably went overboard with those, but I just wanted to implement that **for such a long time**, that I figured I'd put them to good use! Haha.

<sub>_\* I was unpleasantly suprised to discover that many tag editing libraries **offer a markedly worse user experience** than what plain `<input/>` fields already do._</sub>


  [1]: https://i.imgur.com/Ytl1gCu.png
  [2]: http://bevacqua.github.io/woofmark "Woofmark demo on GitHub pages"
  [3]: http://bevacqua.github.io/woofmark "Woofmark demo on GitHub pages"
  [4]: http://stompflow.com "Stompflow: Hassle-free project management"
  [5]: http://stompflow.com "Stompflow: Hassle-free project management"
  [6]: https://github.com/bevacqua/insane "bevacqua/insane on GitHub"
  [7]: https://github.com/bevacqua/megamark "bevacqua/megamark on GitHub"
  [8]: https://github.com/bevacqua/crossvent "bevacqua/crossvent on GitHub"
  [9]: http://jquery.com/ "jQuery: write more, do less"
  [10]: https://github.com/tmpvar/jsdom "tmpvar/jsdom on GitHub"
  [11]: http://nodejs.org/ "nodejs.org"
  [12]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [13]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [14]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [15]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [16]: /articles/designing-front-end-components "Designing Front-End Components"
  [17]: https://github.com/mishoo/UglifyJS2 "mishoo/UglifyJS2 on GitHub"
  [18]: https://github.com/substack/tape "substack/tape on GitHub"
  [19]: https://github.com/substack/node-browserify "substack/node-browserify on GitHub"
  [20]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [21]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [22]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [23]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [24]: https://i.imgur.com/9qZOsZi.png 
  [25]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [26]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [27]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [28]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [29]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [30]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [31]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [32]: https://github.com/bevacqua/horsey "bevacqua/horsey on GitHub"
  [33]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
  [34]: https://i.imgur.com/srcrxSY.png
  [35]: http://bevacqua.github.io/horsey "horsey on GitHub Pages"
  [36]: https://github.com/bevacqua/woofmark "bevacqua/woofmark on GitHub"
  [37]: http://bevacqua.github.io/dragula "Dragula on GitHub Pages"
  [38]: https://github.com/bevacqua/domador "bevacqua/domador on GitHub"
  [39]: https://github.com/bevacqua/woofmark/blob/b49c60537ed7ce71ffb27d460bec6e02d538ad14/src/polyfills/getSelection.js "woofmark.getSelection on GitHub"
  [40]: http://codepen.io/bevacqua/full/bdxajE/ "CodePen.io"
