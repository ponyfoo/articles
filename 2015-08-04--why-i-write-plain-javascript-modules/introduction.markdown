Our web needs **better primitive** libraries. We've been relying for too long -- **far too long** -- on jQuery. Most popular UI components are tied to jQuery, part of a comprehensive framework -- and it's usually hard to extract the component as a standalone library. Nowadays we may not develop _as many jQuery plugins_ as we've used to, but the situation is **far more severe** now.

Today, many popular libraries -- _UI components or otherwise shiny client-side JavaScript things_ -- are bound to the author's preferred coding style. Thus, we create things like [`react-dnd`][1], [`angular-dragdrop`][2], or [`backbone-draganddrop-delegation`][3]. Out of the three, _none_ are backed by a library providing the primitives into drag and drop, without the **tight framework bindings** -- such as [`dragula`][4].

> It all comes down to [composability][5] and _portability_.

[1]: https://github.com/gaearon/react-dnd
[2]: https://github.com/codef0rmer/angular-dragdrop
[3]: https://github.com/christianalfoni/backbone-draganddrop-delegation
[4]: https://github.com/bevacqua/dragula
[5]: /articles/composable-ui
