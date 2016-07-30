# UI

[Dragula](https://github.com/bevacqua/dragula) is my most recent creation. It provides a simple API to manage _drag & drop_ across containers. You can set it up to remove elements from the DOM when they’re dropped outside of the desired containers, tell it to move or clone elements, and listen for events at a few crucial points in the drag & drop lifecycle. It handles sorting, displays a nice shadow for visual feedback that follows the mouse cursor around, and you also get a shadow on the containers that shows where an element would get dropped. It’s also drastically small and simplistic.

[![See a demo of Dragula online](https://github.com/bevacqua/dragula/raw/master/resources/demo.png)](http://bevacqua.github.io/dragula/ "Try out Dragula online!")

[Rome](https://github.com/bevacqua/rome) is a full featured date _and time_ picker that you can attach to an input or render inline. It supports **IE7+** out the box, and comes with minimal styling so that you can style it to match your designs. It has extensive validation and formatting support and also comes with a flexible API to cater for many different use cases. Calendars can also be linked to each other to validate date ranges.

[![See a demo of Rome online](https://cloud.githubusercontent.com/assets/934293/3803583/387125ea-1c1c-11e4-974e-467984e4d1f0.png)](http://bevacqua.github.io/rome/ "Try out rome online!")

[Insignia](https://github.com/bevacqua/insignia) is an enhancement for text based inputs where the user will get **a fancier representation of tags**. They'll still be able to edit tags using plain text input, and you’ll be able to render tags however you want, matching the styling tags have in other parts of your site.

[![See a demo of Insignia online](https://camo.githubusercontent.com/2c61248fb1272df8a619c95c7acfdb8a3f7193bd/687474703a2f2f692e696d6775722e636f6d2f6d6879334676392e706e67)](http://bevacqua.github.io/insignia/ "Try out Insignia online!")

[Horsey](https://github.com/bevacqua/horsey) is **an autocompletion component** that allows you to lazy load suggestions. Given that it only cares about providing relevant suggestions, it plays really well with others. It can be paired with [Insignia](https://github.com/bevacqua/insignia) to suggest tags. You can customize the appearance of suggestions as well as the values that'll be sent to the input. You decide whether the suggestions should be appended to the input value or replace it altogether, among other options.

[![See a demo of Horsey online](https://camo.githubusercontent.com/ba466d12f9a3175daa526c67b2cdf7f0e628df81/687474703a2f2f692e696d6775722e636f6d2f696d44464330432e706e67)](http://bevacqua.github.io/horsey/ "Try out Horsey online!")

[Hint](https://github.com/bevacqua/hint) is a minor component to display **pretty tooltips using pure CSS**. You can add a small piece of JavaScript to add a few more improvements to the tooltips such as adjusting them to the viewport size and fading them in with a nice transition animation.

![A hint tooltip](https://camo.githubusercontent.com/e7fef05529a194b8238efd6a7df9f0a16c65daef/687474703a2f2f692e696d6775722e636f6d2f454650356a34452e706e67)

[Flexarea](https://github.com/bevacqua/flexarea) creates a barely useful full-width drag handle that lets you resize a text area or any other DOM element. _It was one of the first small components [I developed when redesigning Pony Foo](/articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo")_.

# MVC

[Taunus](https://github.com/taunus/taunus) is **an MVC library that enables shared rendering and empowers progressive enhancement**. The server-side rendered application is immediately ready for human consumption and deferring JavaScript is recommended. It’s highly modular and encourages you to keep your views and controllers in individual files. Taunus has solutions for versioning a single page application even when the user doesn’t navigate away after a deployment, lazy loading of views, caching, and prefetching when links are hovered over. [Learn more about Taunus by reading through its documentation](http://taunus.bevacqua.io/).

[Ruta3](https://github.com/bevacqua/ruta3) is the **client-side router** used by Taunus. It’s really small and flexible. You can pass in routes and handlers, and then figure out what handler should be used for a given URL. The routes support named parameters, wildcards, and optional parts.

# Utilities

[Contra](https://github.com/bevacqua/contra) is a utility library similar in spirit to [async](https://github.com/caolan/async) but tuned for cross browser support and a tiny footprint. Contra provides the usual asynchronous flow control suspects such as serial or concurrent flows, as well as **an asynchronous queue implementation that actually underlies most of Contra**. It also comes with a tiny event emitter implementation that’s used in many of the other libraries I described here.

[Fuzzysearch](https://github.com/bevacqua/fuzzysearch) is the default engine used by [Horsey](https://github.com/bevacqua/horsey) to provide relevant search results. **It partially matches user input to produce interesting results**. It’s blazing fast and hilariously small.

[But](https://github.com/bevacqua/but) is a functional utility that helps you **omit wrapper functions** to _“chew up”_ arguments.

[Estimate](https://github.com/bevacqua/estimate) **calculates the time it would take to read the contents of a DOM element** or piece of text. You can also use it to figure out how much of that element has been read, according to the user’s scroll position.

# DOM

[Diferente](https://github.com/bevacqua/diferente) leverages the [virtual-dom](https://github.com/Matt-Esch/virtual-dom) module to easily diff a DOM element and an updated piece of HTML.

[Bullseye](https://github.com/bevacqua/bullseye) is a tiny library that’s able to **inform you of the viewport position for the text selection caret or a particular DOM element**, and report on changes to that position. This is used by both Horsey and Rome, among others, to display elements in the correct place on screen.

[Kanye](https://github.com/bevacqua/kanye) makes it **easy to deal with keyboard shortcuts** by allowing you to declare them in a human way. For example you can specify a shortcut with just `'cmd+shift+x'` and a callback. It allows you to group shortcuts in different contexts which you can then remove at once. This comes in handy when dealing with view-based shortcuts in single page applications.

[Poser](https://github.com/bevacqua/poser) is an interesting experiment where you **get a reference to objects in different execution contexts**. In the browser, you can use Poser to get a reference to an `Array` from an `<iframe>`, and then extend that `Array` to create libraries that **extend true native array functionality**, without interfering with the `Array` everyone else is using.

[Sektor](https://github.com/bevacqua/sektor) is a **tiny DOM selection library** with an API that’s identical to [Sizzle](http://sizzlejs.com/). The difference is that Sizzle implements an entire selection engine in pure JavaScript, while Sektor mostly leverages `document.querySelectorAll`. Sizzle’s implementation fixes bugs that hardly ever come up in the context of regular web applications, and comes at the price of a much larger footprint than that of **Sektor, which sits at 824 bytes**.

[Dominus](https://github.com/bevacqua/dominus) leverages the last two libraries that I’ve mentioned, [Poser](https://github.com/bevacqua/poser) and [Sektor](https://github.com/bevacqua/sektor), to provide **a lean jQuery-like API**. It differs from jQuery in that it extends real `Array` objects with `poser`. It provides more consistency in regards to API method naming and expected outputs.

[Crossvent](https://github.com/bevacqua/crossvent) is a tiny library to bind, unbind, and synthesize DOM events. It’s used by Dominus to provide **cross-browser event handling**.

[local-storage](https://github.com/bevacqua/local-storage) provides a simple **cross-browser solution to access the `localStorage` API**. It provides a stub even when `localStorage` isn’t available, and simplifies the API to handle `'storage'` events.

[insert-rule](https://github.com/bevacqua/insert-rule) is a tiny cross-browser library that allows you to **insert CSS rules into the document programatically** using JavaScript.

[Sell](https://github.com/bevacqua/sell) makes **text selection simpler across browsers**. It returns an object indicating the start and end positions of the text selection, and it also allows you to use those objects to update the text selection on an input.

# Markdown

[Megamark](https://github.com/bevacqua/megamark) builds upon [markdown-it](https://github.com/markdown-it/markdown-it) adding built-in syntax highlighting, HTML sanitization, a trimmed down client-side implementation, a simplified tokenization API, and prettified text.

[Insane](https://github.com/bevacqua/insane) is the **HTML sanitizer** used by Megamark. It follows a whitelist-based approach where you can specify which tags, attributes, CSS class names, and URL schemes are allowed. Everything else gets stripped away. You can also define a `filter` that determines whether a DOM element should be discarded based on any conditions you need.

[Domador](https://github.com/bevacqua/domador) **parses a DOM tree or HTML into Markdown**. It is intended mostly for use when developing two-way Markdown editors that are able to deal with Markdown or HTML input. You can define custom transformers so that extensions to the Markdown language are properly parsed back into the syntax you originally intended.

# Awkward Microsites

[Hubby](https://github.com/bevacqua/hubby) is a single page site that **shows some more information about a GitHub user than their GitHub profile** will show. Mildly useful when stalking developers with tons of starry repositories. Here’s an example: [@sindresorhus](http://bevacqua.github.io/hubby/?sindresorhus).

[Cube](https://github.com/bevacqua/cube) is **a ridiculous game** which will most definitely make you throw up. Includes sounds, trashy seizure-inducing graphics, and a thoroughly unenjoyable gaming experience. _Hacked together at CampJS._

**Wheew.** That’s quite a few modules. Here's hoping you've found _something_ you could use in there. As a bonus for getting this far, consider checking out [js](https://github.com/bevacqua/js) and [css](https://github.com/bevacqua/css), a pair of quality guides I've put together for developing better front-end codebases.
