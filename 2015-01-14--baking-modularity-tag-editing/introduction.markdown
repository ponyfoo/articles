Turns out, all libraries out there _(google for ["tag input javascript"][1] if you're curious)_ that do this have at least one of these flaws.

* Feature bloat
* Human unfriendly
* Addiction to dependencies

Instead of creating separate components for, _say_, tag input and autocompletion, most of these libraries try to do it all, and fail miserably at **doing any one thing well**. This is the _defining characteristic_ of modular development, where we define a _(narrow)_ purpose for what we're building and we set out to build the best possible solution to meet that purpose.

> Multi-purpose libraries end up doing a lot of things I do not need or want, while single-purpose components typically do exactly what you need.

Being user friendly is why we build these libraries, and yet I couldn't find a single library that I would've used over a simple `<input type='text' />` and telling my humans to enter tags separated by a single space. The vast majority of these presented inconveniences such as being **unable to walk around the input** like we do in an input tag, insert tags in any position, or **not even allowing humans to edit tags** once they receive the _"this is a tag"_ treatment.

> Human-facing interface elements _must_ be sensitive to the needs of the human. Sometimes we have to wonder if a component is even worth the trouble. Wouldn't a simple `<input>` be **more amicable** than an inflexible _"tag input"_ solution? Is the component more useful to your humans?

Finally, most of these libraries tie themselves to a dependency. If I really liked one of them, I probably would've needed to commit to jQuery myself. Or Angular. Or React. It doesn't matter what their _"big dependency"_ is, the point is that it **constraints** the consumer from being able to easily **port the component to different projects** with other assortments of frameworks or libraries.

> The fact that most of them count jQuery among their dependencies leaves much to be desired.

This article will explore the approach I took to build [Insignia][2] in just one day, as I think you may apply some of these concepts to any development you do on a daily basis.

[1]: https://www.google.com/search?q=tag+input+javascript
[2]: https://github.com/bevacqua/insignia
