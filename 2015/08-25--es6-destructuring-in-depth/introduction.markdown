This article **warns about going overboard** with ES6 language features. Then we'll start off the series by discussing about Destructuring in ES6, and when it's most useful, as well as some of its gotchas and caveats.

## A word of caution

When uncertain, chances are <mark>you probably should default to ES5 and older syntax instead of adopting ES6 just because you can</mark>. By this I don't mean that using ES6 syntax is a bad idea -- quite the opposite, see I'm writing an article about ES6! My concern lies with the fact that when we adopt ES6 features we must do it because **they'll absolutely improve our code quality**, and not just because of the _"cool factor"_ -- whatever that may be.

The approach I've been taking thus far is to write things in _plain ES5_, and then adding ES6 sugar on top where it'd genuinely improve my code. I presume over time I'll be able to more quickly identify scenarios where a ES6 feature may be worth using over ES5, but when getting started it might be a good idea _not_ to go overboard too soon. Instead, carefully analyze what would fit your code best first, and **be mindful of adopting ES6**.

> This way, you'll **learn to use** the new features in your favor, rather than just _learning the syntax_.

Onto the cool stuff now!
