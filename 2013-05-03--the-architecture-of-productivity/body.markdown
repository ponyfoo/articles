While it's true that end user experience is the _single most important aspect_ in software, it's undeniable that there are other aspects of software that should be treated carefully as well. I don't subscribe to the theory that "anything you do beyond doing the bare minimum in order to solve the problem is utterly worthless", which Ryan hints at.

When developing an application, you could always [over-engineer](http://en.wikipedia.org/wiki/Overengineering "Over-engineering definition"), which is bad, and where I'd agree with Ryan. There are lots of principles you should try and follow here. [YAGNI](http://en.wikipedia.org/wiki/You_aren't_gonna_need_it "You ain't gonna need it") and [KISS](http://en.wikipedia.org/wiki/KISS_principle "Keep it simple, stupid") are important ones in regards to over-engineering.

![over-engineering-meme.jpg][1]

But _not everything is over-engineering_. Picking and tuning a text editor (in fact, my next blog topic), configuring a more seamless [build](/search/tagged/build "search posts tagged build") process, and learning the ins and outs of your everyday toolset, or more generally things that not _likely_ to change, are all very effective ways to boost your development productivity. And, to me, it doesn't _just end_ with configuring your environment, although it's perhaps the most significant productivity boost you could earn.

Writing clean, _though concise_, application architectures can help to achieve the simplicity demanded by large software projects, while not being strictly directed towards solving the problem at hand. Clean architectures go a tremendous way towards the goal of positive user experiences, speed up development cycles by removing redundancy ([DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself "Don't repeat yourself")), and ensuring that no unexpected bugs are introduced, by making the application more easily testable.

On the subject of not adding to the problem, I couldn't agree more. but that's something that's easily avoidable for the _vast majority_ of software developers out there, who aren't working on languages, compilers, operating systems, or in a sufficiently large company that warrants creating a widely used DSL (such as [FQL](https://developers.facebook.com/docs/reference/fql/ "Facebook Query Language")).

I would reword his conclusion as:

> The single most important aspect in software development is end-user experience, but productivity in day-to-day software development is also **crucial** if you are expected to meet deadlines _today and in the future_. The software development experience might be just as important.

Below is the post [Ryan](https://github.com/ry "Ryan Dahl") originally wrote.

> I hate almost all software. It's unnecessary and complicated at almost
> every layer. At best I can congratulate someone for quickly and simply
> solving a problem on top of the shit that they are given. The only
> software that I like is one that I can easily understand and solves my
> problems. The amount of complexity I'm willing to tolerate is
> proportional to the size of the problem being solved.
> 
> In the past year I think I have finally come to understand the ideals
> of Unix: file descriptors and processes orchestrated with C. It's a
> beautiful idea. This is not however what we interact with. The
> complexity was not contained. Instead I deal with DBus and /usr/lib
> and Boost and ioctls and SMF and signals and volatile variables and
> prototypal inheritance and _C99_FEATURES_ and dpkg and autoconf.
> 
> Those of us who build on top of these systems are adding to the
> complexity. Not only do you have to understand $LD_LIBRARY_PATH to
> make your system work but now you have to understand $NODE_PATH too -
> there's my little addition to the complexity you must now know! The
> users - the one who just want to see a webpage - don't care. They
> don't care how we organize /usr, they don't care about zombie
> processes, they don't care about bash tab completion, they don't care
> if zlib is dynamically linked or statically linked to Node. There will
> come a point where the accumulated complexity of our existing systems
> is greater than the complexity of creating a new one. When that
> happens all of this shit will be trashed. We can flush boost and glib
> and autoconf down the toilet and never think of them again.
> 
> Those of you who still find it enjoyable to learn the details of, say,
> a programming language - being able to happily recite off if NaN
> equals or does not equal null - you just don't yet understand how
> utterly fucked the whole thing is. If you think it would be cute to
> align all of the equals signs in your code, if you spend time
> configuring your window manager or editor, if put unicode check marks
> in your test runner, if you add unnecessary hierarchies in your code
> directories, if you are doing anything beyond just solving the problem
> - you don't understand how fucked the whole thing is. No one gives a fuck about the glib object model.
> 
> The only thing that matters in software is the experience of the user.

I recommend reading [The Pragmatic Programmer](http://www.amazon.com/dp/020161622X "The Pragmatic Programmer on Amazon"), if you are interested in learning more on how to be productive while staying on target and be a solid software thinker in general.

[![pragmatic-programmer.jpg][2]](http://www.amazon.com/dp/020161622X "The Pragmatic Programmer on Amazon")

Until the next time!

  [1]: https://i.imgur.com/JPsizDt.jpg "Over-engineering is bad"
  [2]: https://i.imgur.com/3W9BJTe.jpg
