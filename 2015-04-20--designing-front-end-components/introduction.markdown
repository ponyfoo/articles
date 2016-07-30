[Dragula][1] had a much faster growth rate than [rome][2] did, and that's mostly due to the fact that there were basically _zero_ proven libraries that **solve drag & drop** _(and drag & drop alone)_ while being flexible and simple. I emphasize the fact that it does _less_ as a feature, and not a problem. In my experience, I've found that the best pieces of code I've ever used tend to follow the UNIX philosophy of **doing one thing very well**. Having that kind of focused approach to front-end modules typically isn't the case. Instead, we often develop solutions that are only useful in our specific use. For example, we may put together an entrenched AngularJS directive that uses other directives or relies on data-binding provided by Angular.

What's worse, most often we don't _just_ limit ourselves to the scope of the libraries we're currently using, but to the specific scope of the one project we're working on. This means that now our library can't be extricated from that database without considerable work. For instance, consider that one time you developed a UI component where the user would type tags in plain text and after typing a delimiter _(a comma or a space)_, they would get visual feedback into the tag being accepted. Something like the screenshot below.

[![Screenshot of Insignia in action][4]][3]

Except for the fact that, unless you found what you needed on the open web in the form of a decoupled module, we sometimes commit the crime of tightly coupling the tagging feature to the framework we're using. Or to a library. Or to our application. Or to a specific part of our application.

It would be best if were able to identify what it is that we're trying to accomplish _(in this case, improve the UX on an input field for tags)_, isolate that into its own module, and come up with the simplest possible API. Coming up with a simple API is often a dreadful task, and in this article I'll try to put together advice and explain my thought process while designing them.

[1]: https://github.com/bevacqua/dragula
[2]: https://github.com/bevacqua/rome
[3]: http://bevacqua.github.io/insignia/
[4]: https://camo.githubusercontent.com/2c61248fb1272df8a619c95c7acfdb8a3f7193bd/687474703a2f2f692e696d6775722e636f6d2f6d6879334676392e706e67
