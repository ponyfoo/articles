The first thing anyone is taught when approaching Angular for the first time is that directives are meant to interact with the DOM, or whatever does DOM manipulation for you, such as jQuery [_(Get over it!)_][1]. What immediately becomes **(and stays)** confusing for most, though, is the interaction between scopes, directives, and controllers. Particularly when we focus on scopes, and start factoring in the advanced concepts: **the digest cycle, isolate scopes, transclusion, and the different linking functions in directives.**

This [_(two-part)_ article][5] aims to navigate the salt marsh that are Angular scopes and directives, while providing an amusingly informative, in-depth read. In the first part, this one, I'll focus on scopes, and the life-cycle of an Angular application. The second part is focused on directives

> The bar is high, but scopes are _sufficiently hard_ to explain. If I'm going to fail miserably at it, at least I'll throw in a few more promises I can't keep!

If the following figure [_(source)_][4] looks unreasonably mind bending, then this article might be for you.

[![mindbender.png][2]][4]

_Disclaimer: article based on [Angular v1.2.10 tree @ `caed2dfe4f`][3]._

[1]: /2013/07/09/getting-over-jquery
[2]: https://i.stack.imgur.com/O1iSG.png
[3]: https://github.com/angular/angular.js/tree/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d
[4]: https://github.com/angular/angular.js/wiki/Understanding-Scopes "Understanding Scopes - Angular wiki on GitHub"
[5]: /2014/02/19/angle-brackets-synergistic-directives "Angle Brackets, Synergistic Directives"
