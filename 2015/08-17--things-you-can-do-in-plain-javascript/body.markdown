## `element.getBoundingClientRect`

You could use  [`getBoundingClientRect`][1] to get the current size of any DOM element, as well as it's current position, relative to the viewport.

In the screenshot below I chose a random DOM element on the Elements tab of Dev Tools. Then I popped open the Console and used `getBoundingClientRect` on `$0` -- which is always bound to the last DOM element selected in the Elements tab.

[![rect.png][2]][1]

In [`dragula`][5], for example, I use `getBoundingClientRect` to figure out [the absolute positioning][4] of the elements that get dragged around by humans. A better example might be [`bullseye`][6] used in [`rome`][7], [`horsey`][8], and [`woofmark`][9] to place elements right below inputs, textareas, or even the text selection caret _(although that one uses `getSelection`, polyfilled by [`seleccion`][10], to get the selection offset)_.

The `getBoundingClientRect` method also has great browser support: **all the way down to IE4**. On _IE8 and older_, the often convenient `width` and `height` properties aren't provided, but they could be inferred as `right - left` and `bottom - top` respectively.

## `document.elementFromPoint`

Another cool piece of the web API that enjoys from broad browser support is [`elementFromPoint`][11]. It can find the topmost DOM element at any `(x, y)` point in the document. If the topmost element is inside an `<iframe>`, the `<iframe>` itself is returned. This method vastly simplifies the code in `dragula`, which needs to figure out what's behind the element being dragged.

The code below hides the element being dragged using a class that sets `display: none !important`, ensuring that the subsequent `elementFromPoint` call returns the element that's below. It surely doesn't come up quite that often, but it's still a nifty hack to leverage!

```js
function getElementBehindPoint (point, x, y) {
  var old = point.className;
  point.className += ' gu-hide';
  var el = <mark>document.elementFromPoint(x, y);</mark>
  point.className = old;
  return el;
}
```

Besides drag and drop situations, this method is often used in integration tests using Selenium, when testers want to click a button that can be found in a precise point in the document. There's probably a few more use cases, but there's generally _way cleaner ways_ to get references to DOM elements you care about.

## `selectionStart` and `selectionEnd`

Whenever we modify user input we need to be careful enough to preserve their text selection. This comes up when we replace tokens such as at mentions [_(see the `<textarea>` example here)_][13], insert images or links in rich text-editing scenarios, and similar situations where you want to manipulate user input while it's being entered.

Both of these properties can be found in input elements. You can read them directly, and you could also change their value, immediately updating the DOM as you're used for other DOM-changing getters and setters like style properties such as `el.style.display`.

If you need [support below IE9][14] you'd have to use something like [`sell`][15], which leverages the TextRange API in those scenarios, and otherwise uses `selectionStart` and `selectionEnd`.

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][3]._ Remember to **subscribe** if you got this far!


  [1]: https://developer.mozilla.org/en/docs/Web/API/Element/getBoundingClientRect "Element.getBoundingClientRect() – MDN"
  [2]: https://i.imgur.com/FO1GqeR.png
  [3]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
  [4]: https://github.com/bevacqua/dragula/blob/8ebbffc7a674234cc55e757155d981dca9ab3288/dragula.js#L463-L469 "getOffset() method in dragula"
  [5]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
  [6]: https://github.com/bevacqua/bullseye/blob/bab4799cdf02e5df2bef81d48faddc75a0b03f6f/bullseye.js#L44 "bevacqua/bullseye on GitHub"
  [7]: https://github.com/bevacqua/rome "bevacqua/rome on GitHub"
  [8]: https://github.com/bevacqua/horsey "bevacqua/horsey on GitHub"
  [9]: https://github.com/bevacqua/woofmark "bevacqua/woofmark on GitHub"
  [10]: https://github.com/bevacqua/seleccion "bevacqua/seleccion on GitHub"
  [11]: https://developer.mozilla.org/en-US/docs/Web/API/Document/elementFromPoint "Document.elementFromPoint() – MDN"
  [12]: https://github.com/bevacqua/dragula/blob/8ebbffc7a674234cc55e757155d981dca9ab3288/dragula.js#L483-L491 "getElementBehindPoint() method in dragula"
  [13]: http://bevacqua.github.io/horsey/ "Horsey autocomplete demo on GitHub Pages"
  [14]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setSelectionRange "HTMLInputElement.setSelectionRange() – MDN"
  [15]: https://github.com/bevacqua/sell "bevacqua/sell on GitHub"
