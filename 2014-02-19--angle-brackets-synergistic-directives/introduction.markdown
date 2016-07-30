If the following figure [_(source)_][1] looks unreasonably mind bending, then this article might be for you.

[![scope.png][2]][1]

_Disclaimer: article based on [Angular v1.2.10 tree @ `caed2dfe4f`][3]._

[][4]

# What the hell is a directive?

Directives are _typically small_ components which are meant to interact with the DOM, in Angular. They are used as an abstraction layer on top of the DOM, and most manipulation can be achieved without touching DOM elements, wrapped in jQuery, jqLite, or otherwise. This is accomplished by using expressions, and other directives, to achieve the results you want.

Directives in Angular core can bind an element property **(such as visibility, class list, inner text, inner HTML, or value)** to a scope property or expression. Most notably, these bindings will be updated whenever changes in the scope are digested, using watches. Similarly, and in the opposite direction, DOM attributes can we "watched" using an `$observe` function, which will trigger a callback whenever the watched property changes.

Directives are, simply put, the single most important face of Angular. If you master directives you won't have any issues dealing with Angular applications. Likewise, if you don't manage to get a hold of directives, you'll be cluelessly grasping at straws, unsure what you'll pull off next. Mastering directives takes time, particularly if you're trying to stay away from merely wrapping a snippet of jQuery-powered code and calling it a day.

> While directives are supposed to do all of the DOM manipulation, **it's [synergy][5] you should be exploiting**, and not jQuery.

Synergy. **Synergy is the long sought-after secret sauce.**

# Secret Sauce: Synergy

[_Synergy_][5] is a term I've became intimate with a few years back, as the [_enthusiastic_ (fat) Magic: The Gathering _(MTG)_ player][6] I used to be. The best decks in _MTG_ often are those where each card in your sixty-card deck is empowered by its relationship with the rest of your deck. In these synergistic decks, you run with a significant advantage: each card you draw has the potential of improving the impact each card in your hand has, and this effect grows exponentially, as you draw more cards. You could say that the two most important factors in building a good deck is card drawing sources, and synergistic potential.

It's not that different with Angular applications. In Angular applications, the more and more you use Angular's inner-mechanisms, such as scopes, events, services, and the different options available to directives, the more synergistic your application will be. Synergy translates into reusability. Highly synergistic components allow you to share them across parts of an application, or even across applications entirely.

Synergy makes Angular applications _work as if magic existed_. **Synergy makes complex interaction feel easy, reasonable, and understandable.** Synergy is what drives _complex interaction into its building blocks_, breaking it down into the essentials, which anyone can understand. Synergy is everywhere, synergy isn't just in code itself, but **you can find it in UX, too.** A synergistic application will feel more natural, easier to use, and more intuitive. You'll feel like _you know the application_, and often guess correctly what the next step will look like, because _the application author cared_ about what you'd think should happen.

In Angular, synergy means being able to build componentized directives, services, and controllers, which can be reused as often as it makes sense for them to be reused. For instance, you might have a simple directive which turns on a class based on a watched scope expression, and I'd imagine that could be a pretty common directive, used everywhere in your application, to signal the state of a particular component in your code. You could have a service to aggregate keyboard shortcut handling, and have controllers, directives, and other services, register shortcuts with that service, rooting all of your keyboard shortcut handling in one nicely self-contained service.

Directives are _also_ reusable pieces of functionality, but most often, **these are associated to DOM fragments, or templates**, rather than merely providing functionality. Markup is equally important in providing synergy, if not even more so. Time for me to give you a deep down dive of Angular directives and their use cases.

[1]: http://docs.angularjs.org/guide/concepts
[2]: https://i.stack.imgur.com/fkWHA.png
[3]: https://github.com/angular/angular.js/tree/caed2dfe4feeac5d19ecea2dbb1456b7fde21e6d
[4]: http://angularjs.org/
[5]: http://en.wikipedia.org/wiki/Synergy
[6]: http://www.wizards.com/Magic/Magazine/Events.aspx?x=mtgevent/gpba08/welcome#9
