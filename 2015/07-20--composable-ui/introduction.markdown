> Before I go any further, [`dragula`][1] now has over 7500 stars on GitHub and was even featured on [The Next Web][2]! _I'll need a few moments_ to wrap my head around **that**! Wow.
> 
> If you've never seen `dragula`, [check out the demo][3] to get an idea of what it does.

Instead of having an autocomplete, a drop down list, a combo box, and a select box, it's nice when you can [do all of that with a single component][4]. That's _reusability_, you get to use the same component for many different use cases. Another form of reusability is **composability**, _the ability to integrate a few different components into a cohesive user experience_. It's an often overlooked factor when developing UI components, but I think **good UI components must be highly composable**.

Composability can also go the opposite direction. Some programs are notoriously modular, and it gets quite a few bits and pieces _(otherwise called "modules")_ to get to something that "works". One such example is [`woofmark`][5]. This library provides an editor on top of browser HTML `<textarea>` elements. I'm not using `woofmark` on ponyfoo quite yet, because I haven't had the time to integrate it here, but I did implement some of the underlying modules. For instance, `woofmark` uses `megamark` under the hood to parse Markdown strings into HTML strings. [Megamark][6] is really an abstraction layer on top of [`markdown-it`][7], which **adds a couple of minor niceties** such as _syntax highlighting_ in code blocks, and the ability to use <kbd><mark\></kbd> DOM elements to <mark>highlight a _"fancy"_ piece of text</mark>. Woofmark also uses [`domador`][8] to convert DOM trees or HTML strings back to Markdown, and that has a extensibility structure in place allowing one to extend their understanding of how HTML should be converted into Markdown. If you had custom _Markdown -\> HTML_ directives, it's only logical that you implement the reverse for a _HTML -\> Markdown_ conversion. With all this converting going on, it's probably [`insane`][9] to try and sanitize the inputs and outputs on your own, so a specialized library takes care of sanitization at the HTML level.

I've already mentioned a few modules, each in charge of a different portion of the rich-editing experience in [`woofmark`][5]. Here's a list of the modules I've mentioned, plus a few others even deeper in the dependency chain.

* [`markdown-it`][7] is one of the **lowest-level** modules used in this use case, and it parses <mark>[CommonMark][10]-compliant</mark> markdown into HTML strings
* [`megamark`][6] is a wrapper around `markdown-it` which ties it with [`highlight.js`][11], a syntax highlighting module and [`insane`][9], the HTML sanitizer
* [`domador`][8] _parses HTML or DOM nodes back into Markdown_, and can be extended to match the extensions provided to `megamark`, so that <mark>the output stays consistent both ways</mark>
* [`woofmark`][5] ties everything together and provides a nice `<textarea>` upgrade that allows entering Markdown, HTML, and WYSIWYG input in exchange for plain Markdown
* Plenty of other low-level modules are at work, such as [`crossvent`][12] for dealing with DOM events in a cross-browser manner; [`he`][13], which deals with unicode; and [`jsdom`][14], which deals with creating a `window` context on the server-side; and **many others**

As you can probably imagine, trying and cramming all of this functionality into a single module would be a dreadful endeavour, not to mention **a waste of productivity, time, and hence, _ultimately_, money**. In contrast, keeping the functionality in separate modules enables us to reuse them across our stack, across our projects, and out on the open-source world, where <mark>people find it way easier to contribute patches</mark> if the code is small, self-contained, and _does one thing well_.

[1]: https://github.com/bevacqua/dragula
[2]: http://thenextweb.com/dd/2015/07/20/less-of-a-drag-maaaaaaaan/
[3]: http://bevacqua.github.io/dragula/
[4]: https://github.com/bevacqua/horsey
[5]: https://github.com/bevacqua/woofmark
[6]: https://github.com/bevacqua/megamark
[7]: https://github.com/markdown-it/markdown-it
[8]: https://github.com/bevacqua/domador
[9]: https://github.com/bevacqua/insane
[10]: http://commonmark.org/
[11]: https://highlightjs.org/
[12]: https://github.com/bevacqua/crossvent
[13]: https://github.com/mathiasbynens/he
[14]: https://github.com/tmpvar/jsdom
