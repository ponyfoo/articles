It'd also be pretty great if you could extend Arrays by yourself in a meaningful way. We all know how preposterous it is to extend the almighty `Array` object. There's whole slew of issues that can arise from doing that, and that's the reason why most of the community has shied away from polluting the global namespace, and more importantly, from polluting the `Object`, `Array`, and `Function` prototypes.

> There is a way to have it both ways. You can extend `Array` all you want, and you can also not touch the `Array` object. **Curious?**

All you need is a bag of tricks, a magic wand, and **another execution context**. Or just ignore my blog post and [run to the source][1], Luke.

[1]: https://github.com/bevacqua/poser
