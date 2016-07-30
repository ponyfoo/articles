For the record, here's the animation. If all is well with the world, you should be able to set it in motion yourself by hovering over the icon at the top of this page, but I'm still adding the animation below -- for posterity.

![The ponyfoo.com logo in all of its animated glory][1]

I put together the logo on CodePen. I find CodePen to be extremely useful when doing this kind of thing, as I don't have to set up any live reloading stuff and I can just focus on the little thing I'm trying to accomplish -- in this case, embellish a logo. [Earlier][2] this year I had turned the logo into an SVG graphic, making for a **much** nicer favicon than I used to have. So I took that, I put it [up on CodePen][3], and added a bunch of styles to animate the logo on hover.

<p data-height="232" data-theme-id="9622" data-slug-hash="EVZLLm" data-default-tab="result" data-user="bevacqua" class='codepen'>See the Pen <a href='http://codepen.io/bevacqua/pen/EVZLLm/'>EVZLLm</a> by Nicolas Bevacqua (<a href='http://codepen.io/bevacqua'>@bevacqua</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

The styles are fairly simple. Each _"pixel"_ in the logo is an SVG `<rect>`. These have an SVG `fill` color, and that's used to determine what color should be animated. The sweep effect is caused by the following bit of code. It adds 10 milliseconds to the total length of the animation, times the index in the DOM tree. That means the first pixel animates in full for `710ms`, the second one takes `720ms` to complete, and so on. It really is very simple, and I love how new patterns unfold as the animation goes on.

```stylus
for offset in (1..107)
  &:nth-child({offset})
    animation-duration 0.7s + offset * 0.01
```

Then I tried adding it to my site.

[1]: https://i.imgur.com/oZRo9PN.gif
[2]: https://twitter.com/nzgb/status/619344789616594944
[3]: http://codepen.io/bevacqua/pen/EVZLLm?editors=110 "See it on CodePen"
