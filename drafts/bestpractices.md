# The Burden of Best Practices #

It's time to crush ancient web ideals. Sure, these techniques still have some value. But, quite probably, not as much so as when they were first introduced. They are just buzzwords. Buzzwords which often _confuse novice developers into oblivion_.

Some of these terms have become so inflated that they get regarded as a best practice, even though it's not always the case. Best practice is _too broad of a concept_.

## Progressive Enhancement ##

[Progressive enhancement](http://en.wikipedia.org/wiki/Progressive_enhancement "Progressive Enhancement definition on Wikipedia") is a technique where you pick a starting point, and build from there. Adding, making sure _nothing breaks in older environments_. All as you _add features_ that are only supported in the shiniest browsers.

Theoretically, this is _awesomesauce_. You get to support every browser, regardless of version, _regardless of JavaScript!_ You could rule the world!

You need to know where to draw the line, though. Going _all the way down_ to supporting **no JavaScript** for everyone, you're gonna have _a bad time_.

> If you're like me, _you'd probably want to_ provide accesibility all the way down to `curl`. That doesn't mean you _have to_, it _doesn't mean that **you should**_.

That being said, you should probably support a few versions of IE you're _not particularly fond of_, but only _if your user-base demands it_.

> Do you have any analytics telling you a significant portion of your customer-base runs around in naked browsers with no JavaScript?  
> _Analize your audience_, and figure out _their_ needs.

The problem with progressive enhancement, is that, even though it looks good on paper, it doesn't really provide much added value to your solutions. Unless you really have _a need_ for progressive enhancement, I'd tell you to take it _with a pinch of salt_.