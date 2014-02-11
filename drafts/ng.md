# Angle brackets, rifle scopes

**Angular.js** presents a remarkable number of interesting design choices in its code-base. A particularly interesting case in point is the way in which directives are compiled, and behave.

The first thing anyone is taught when approaching Angular for the first time is that directives are meant to interact with the DOM, or whatever does DOM manipulation for you, such as jQuery [_(Get over it!)_][1]. What immediately becomes **(and stays)** confusing for most, though, is the interaction between scopes, directives, and controllers. Particularly when we focus on scopes, and stark factoring in more advanced use cases such as isolate scopes, transclusion, and the different linking functions in directives.

This article aims to navigate the salt marsh that are Angular scopes, while providing an amusing and informative, in-depth read.

> The bar is high, but scopes are _sufficiently hard_ to explain. If I'm going to fail miserably at it, at least I'll throw in a few more promises I can't keep!

  [1]: /2013/07/09/getting-over-jquery "Getting Over jQuery"

[![angularjs.png][1]][2]

...


scopes
isolate
child

directive
pre
post(default)

transclusion






Please comment on any issues regarding this article so _everyone can benefit_ from your feedback!

  [1]: http://i.imgur.com/LSVpcm1.png
  [2]: http://angularjs.org/ "Angular.js"
