[![allthingsdigital.png][1]][2]

There's always been a long-standing fallacy that sites must look the same on every browser. Maybe that's the reason why it took us so very long to adopt responsive web design. It may be here, at **the intersection between design/UX and performance**, that we may find our answers. After all, if you look to the corners of the web that actually care about both parties, their web experiences are largely as enjoyable as their mobile counterparts. See [Twitter][3] or [The Guardian][4] for reference.

Everything is relative. It's all about context. It's all about priorities. In the world of development, it's all about productivity. We seem to be perfectly okay with disregarding performance in favor of productivity. On the surface, this is reasonable, nobody in their right mind would choose Assembler over JavaScript because of performance, right? Well, yes. But we also have to draw a line at some point. If _slow sites take 8 seconds to load_, and Facebook can bring that down to "zero" by prefetching and whatnot, that's great, from Facebook's business point of view.

[![facebookinstant.png][5]][6]

<sub>Not even a minute for the landing page of a marketing microsite, not bad! (I'm on a hotel's wi-fi in Australia, but that shouldn't be a huge deal, right?)</sub>

Except no reasonable site should ever take 8 seconds to get to their content. I've recently gave a talk on the performance optimization subject at [CampJS][7], and [here's my talk slides][8]. I've also compiled the talk down into [a self-guided workshop called `perfschool`][9], with _ten different exercises_, that I encourage you to try and walk through from the comfort of your home. If you still think you can't get your site to be drastically faster, I encourage you to check out the content churned by all of the authors I've mentioned above, since they care about performance quite a bit and they always produce quality articles that result in **actionable performance gains**. [@scottjehl][10], [@paul_irish][11], [@addyosmani][12], and [@igrigorik][13] are also some other people to track in terms of better understanding performance, and also have a solid grasp on user experience and design.

There's dozens of web specification drafts being worked on, and the fact that the web is the largest available runtime doesn't quite make it a piece of cake to quickly churn out updates -- the rate at which innovation on the web is occurring, regardless, is nothing short of astonishing.

Changing the public's perception on the web being inherently slow is going to be a long and winding road. Most of the way, though, it's going to be about not comparing the web to mobile anymore. It's not like Facebook Instant Articles or mobile apps do anything fundamentally different than relying heavily on the web to operate. The same web we've already wrote off as "too slow to be in the picture".

The best ways of improving the situation, as well eventually come to see, are to [go back to the server][14], adjust [UX for humans][15], [measuring performance][17], [optimizing the critical rendering path][16], following [style guides][18], and writing all the modules.

Content is king. Defer the rest, and [stop breaking the web][19].

[1]: https://i.imgur.com/XgUVDQo.jpg
[2]: https://www.flickr.com/photos/merlin/sets/72157622077100537/ "Noise-to-Noise Ratio - flickr.com"
[3]: "Twitter.com"
[4]: http://www.theguardian.com "theguardian.com"
[5]: https://i.imgur.com/GdeYiIe.png
[6]: http://instantarticles.fb.com/ "Facebook 'Instant' Articles"
[7]: http://v.campjs.com/#high-performance "CampJS V"
[8]: https://speakerdeck.com/bevacqua/high-performance-in-the-critical-path "High Performance in the Critical Path"
[9]: https://github.com/bevacqua/perfschool "bevacqua/perfschool on GitHub"
[10]: https://twitter.com/scottjehl "Scott Jehl on Twitter"
[11]: https://twitter.com/paul_irish "Paul Irish on Twitter"
[12]: https://twitter.com/addyosmani "Addy Osmani on Twitter"
[13]: https://twitter.com/igrigorik "Ilya Grigorik on Twitter"
[14]: /articles/server-first-apps "Server-First Apps on Pony Foo"
[15]: /articles/adjusting-ux-for-humans "Adjusting UX for humans on Pony Foo"
[16]: /articles/critical-path-performance-optimization "Critical Path Performance Optimization on Pony Foo"
[17]: /articles/measure-optimize-automate "Measure, Optimize, Automate on Pony Foo"
[18]: https://github.com/bevacqua/js "bevacqua/js JavaScript Quality Guide on GitHub"
[19]: /articles/stop-breaking-the-web "Stop Breaking the Web on Pony Foo"
