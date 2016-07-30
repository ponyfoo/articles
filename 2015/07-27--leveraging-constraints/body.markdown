A few months back, [Amanda][2] pointed out the [power of constraints][3], and **it _felt_ obvious**. Yet, it has to be pointed out. Then, it <mark>_becomes_</mark> obvious. If you look at great works of art or design, constraints are, for the most part, what drove these artists. In terms of painting -- _or systems design_ -- It's not about what more you can add to your work, or what else you can take out.

#### It's mostly about _what you let yourself leave out_, in the first place.

Consider performance and the [interesting concept of performance budgets][4], where you define a hard lower boundary for how responsive your app or service should be. Here we are defining constraints such as _"the total page weight must be **under 1MB** for initial page load"_ or _"requests against the landing page must attain a **SpeedIndex** score under 1000"_. Consider design and constraints such as _"we must not use images for layout"_, _"we must pick, at most, two custom fonts"_, or _"all icons must be `32px` by `32px` in size"_. These are not exactly orthogonal concepts, quite the opposite actually. Design must be tackled with performance in mind if are to take performance seriously. We need to work together with our design teams so that we don't end up slaves to multiple font downloads _(or even worse, scripts)_.

> **It's not so much about the web being slow as it is us that are making it slow by not enforcing enough constraints.**

Constraining yourself on purpose is not even only useful when you're writing _(prose)_, designing, or defining performance expectations! It's also great when writing code -- a very different form of writing.

Constraints in code may crop up as "using only ['the good parts'][6] in JavaScript", following [style guides][5], developing code in [small modules][7] and [writing tests][8] for it. Similar kinds of constraints can be seen everywhere in the world of software development, and the universe at large.

What kind of constraints do you set upon yourself? Which ones do you think work really well for you?

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][1]._ Remember to **subscribe** if you got this far!

[1]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
[2]: https://twitter.com/amandaglosson "@amandaglosson on Twitter"
[3]: https://www.youtube.com/watch?v=bKkYcetGWjA "Constraints are not Compromises — talk by @amandaglosson at JSFest Oakland"
[4]: /articles/measure-optimize-automate "Measure, Optimize, Automate on Pony Foo"
[5]: https://github.com/bevacqua/js "bevacqua/js on GitHub"
[6]: http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742/ "JavaScript: The Good Parts"
[7]: /articles/great-web-module-compendium "The Great Web Module Compendium on Pony Foo"
[8]: /articles/testing-javascript-modules-with-tape "Testing JavaScript Modules with Tape"
