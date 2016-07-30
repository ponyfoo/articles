In this article well get an in-depth look at these three modules _([tape][1], [proxyquire][2], and [sinon][3])_, learn why they are better than the competition, how to use them, what each of them is good for, and how they complement each other to provide the ultimate _"testing harness"_, figuratively speaking, **no test harness is actually needed!**

While everything else in the JavaScript universe seems to be moving at blisteringly fast speeds, _and accelerating_, testing is in this weird place where globals are mystically okay. Frameworks like [mocha][4] and [jasmine][5] dominate the field, while they require a test harness, **litter the global object with variables**, and generally go against established best practices. I've always found it kind of _really_ weird that test frameworks, which are supposed to be used to _improve the code quality_ around a codebase, may end up doing the exact same opposite by encouraging the usage of globals and awkward monkey patching.

Nevertheless, let's jump into the awesome world of testing with [tape][1], [proxyquire][2], and [sinon][3], right after a brief note about the survey I ran last monday.

> ## A Quick Note About the Survey
> 
> **Many _thanks_** to everyone who took the time to answer [the survey][6]. You gave me a few ideas about the kind of content I could write, and I also got some very useful insight into what people reading this blog enjoy _(and dislike)_ the most.
> 
> I suppose the results themselves aren't that interesting to anyone that's not myself, but [here they are anyways][7], for the sake of transparency. In general, people would like to learn more best practices, guidelines, code quality, tutorials, and in-depth articles, and not so much about rants.
> 
> [![What type of content is liked the most][8]][7]
> 
> _I'll do my best to keep the rants in check, but keep in mind the tagline of this blog!_

[1]: https://github.com/substack/tape
[2]: https://github.com/thlorenz/proxyquire
[3]: https://github.com/cjohansen/Sinon.JS
[4]: https://github.com/mochajs/mocha
[5]: https://github.com/jasmine/jasmine
[6]: https://docs.google.com/forms/d/1pccaq_Tq0QSGKbP9czdEq0uPbFzthKmBY1Kvkjl2Slc/viewform
[7]: https://docs.google.com/forms/d/1pccaq_Tq0QSGKbP9czdEq0uPbFzthKmBY1Kvkjl2Slc/viewanalytics
[8]: https://i.imgur.com/Dgw2mxI.png "Screen Shot 2015-07-06 at 21.19.30.png"
