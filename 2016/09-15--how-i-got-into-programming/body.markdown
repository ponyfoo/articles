# The Early Days

For my last 3 years of high school I went to a school where I could specialize in Computer Systems. By this time I was 16. They taught a bunch of programming languages _-- VB6, C, Java, Pascal, ASP, little bits of web development --_ but more importantly things like algorithms and program design.

I cloned Paint using VB6. I had fun cloning things. In order to draw freely with the pencil, at first my Paint just drew a dot whenever it detected a mouse movement. This was really bad not only for performance reasons, but also due to the fact that computers were so slow back then that VB6 *didn't even bother* reporting every single mouse move event, so the pencil drawings were more like dotted lines. I started tracking state and then each point when using the pencil was drawn as a line between the last point and the current point. The lines looked pretty smoothly, and I was happy about my discovery. Then I went on and opened the real Paint, started drawing by quickly moving the mouse, and figured out that they did the same as I had done! That was something I loved about my Paint clone. Making it helped me figure out how things worked, which is always a rewarding endeavor.

Then nobody believed *I* had actually *made* the Paint clone. They thought I just downloaded it, even when I showed them the *really-important* "About" page showing I was the author. Deep down, I was **proud** they didn't believe I made it.

A while later I made a clone of Snake -- the one where you eat dots and your snake grows until it tries to eat itself. That one went to a science fair they had every year at my school. It was fun seeing people try it out. I had made it.

For the last science fair we made a small collaborative project together with the electronics guys. They handed up a micro controller card and the specs. We then took the motor of a ceiling fan, a baby tub, and some other components and made a prototype for a "remote-controlled swimming pool". I didn't have much to do with the electronics side, but I made the website. It was just an ASP website that took commands and transmitted them to the micro controller connected to that same computer. We brought two computers to the fair, to demonstrate it was actually remote, but nobody seemed to care. I did, it was my favorite part. Anyways, the site allowed you to open or close a ceiling for the swimming pool. The idea was you could keep the pool closed during rains and open it up on sunny days. The site also tracked _-- thanks to some sensors connected to the micro controller --_ the temperature and pH for the pool. You could turn on the heat when the temperature was low, and there was an alert when pH changed considerably. It was a super fun project.

# Ultima Online --- C# and a real job™

Ultima Online (UO), a massively multiplayer online role-playing game (no wonder they abbreviate that as MMORPG), wasn't any different. I played in a local server that turned out to use an [open-source][ruo] implementation of the server written entirely in C#, all the way down to networking packets. The administrators, who had no programming experience, slowly started trusting me to handle minor bug fixes by literally emailing source code files back and forth. I was hooked. C# was a wonderful, expressive language, and the open-source software for the UO server was very amicable and inviting — you didn't even need an IDE (or even know what that was) because the server would compile script files dynamically for you.

![Ultima Online. You could use the official client, a modified client, the official server, or an open-source server. What a game!][uopic]

You would be essentially writing a file with 10-15 lines in it, inheriting from the Dragon class, and adding an intimidating text bubble over their head, or override some method so that they'd spit more fire balls. You'd learn the language and it's syntax without even trying, just by having fun!

> You can check out the [RunUO repository on GitHub][ruo], although the project isn't maintained anymore.

After reaping through some C# manuals, I went beyond scripting dragons and started doing more interesting stuff. My two favorite projects were an automated duelling system, and a new spell system.

The administrators would on a somewhat regular basis, host duelling tournaments. They ran everything by hand: they created a couple of portals in popular cities, and asked people to go through them. Then they'd ask people to send them messages if they wanted to participate or just watch. Then they figured out the brackets for the tournament and kept track of who won or lost. Then they manually moved each player into the duelling zone and out of it. I automated all of this by making a few in-game items and writing a lot of code. It was so much fun. I kept drawing brackets to test out my algorithms and ran them in my mind. The ability to run one of these tournaments whenever I wanted and with zero manual action was so relieving.

The script even had fireworks when the tournament ended! 🎉

Later, [I open-sourced that duelling system][oss]. This was 2006, so my version of open-source back then was opening up a thread on the forums for the server, _(which was on SVN back then)_, and uploading a boatload of documentation and `.rar` files. After more than a month's worth of work, the first comment was a bit underwhelming:

> *"Good job but this thread should be called Tournament System bec its for events."*.

Welcome to open-source!

Another thing I open-sourced, which was far more exciting, was a new spell system: Spellweaving. Ultima Online had a few spell systems *(Magery, Necromancy, etc)*. The real game servers always got the latest patches, but we had to implement these things on our own for the open-source server. Nobody had ever gotten around to implementing Spellweaving, mostly because nobody had the time. I logged into OSI *(the official game server)* and after around a week of research I was somehow able to implement the entire Spellweaving skill from scratch. This was super fun and exciting. It eventually made its way onto the [core RunUO codebase][sw], but not before I had it on my own game servers exclusively for a few months, because I wanted players on my server to savor Spellweaving exclusively for a while.

> Eventually, a friend revealed that I could make a living out of writing C# code *— “You know, people actually pay you to do that”*. That's when I started developing websites again, except I wasn't just using Front Page and piles of `<blink>` and `<marquee>` tags or Java applets, just for fun, anymore.

# A Web Developer Arises

As a web developer, the first few things I worked on were on PHP and C#. The PHP job was before I knew C# was a thing. Imagine that? *Such disconnect.* How could I not even know C# was a serious language people paid you to work with?

**I was just so in love with the C# programming language I couldn't even fathom people actually paying for you to write code in C#.**

Anyways, my PHP job was not interesting. So I quit for a job that paid even less, but where I could use C#.

When I started working with ASP.NET, I had to build a CMS on my own, which was pretty fun. I had one of those "one pixel to the left" kind of bosses, and left promptly after I shipped the first version of the CMS, but I did learn a ton about design at that job. Not from my boss, obviously. He didn't even want me to "commit" to jQuery, because it didn't seem like such a safe bet. This was like 7 years ago, jQuery was **all the rage** back then. I went for jQuery anyways. He couldn't get enough of those sliding animations. I did not like my boss.

After that job I went to a consulting company, where some serious C# and .NET development happened. I worked with ASP.NET MVC and did a ton of front-end work. Nobody at the company -- except for maybe a couple guys -- seemed to like front-end development. I did not understand. JavaScript was fun. It was certainly not C#, but it didn't need to be so complicated either. For a time, I was the go-to front-end guy at this company and I really enjoyed that.

Two years later, **I spent literally a week configuring my development environment for a new project** with a big player in the music industry, and decided it was time to quit.

I had a short stint at another consulting company where I also spent time with.NET technologies, WebSockets, and C#. I saw people using Sublime Text. I was not impressed. I was a man of an IDE. Why would you not use Visual Studio? What even was a terminal prompt, anyway?

Then Node.js piqued my interest.

# The Pragmatic Programmer --- Node.js, and Pony Foo 🦄

A few years ago I read [The Pragmatic Programmer][pp], and something clicked inside me. The book has an assortment of solid advice, and I can't recommend it highly enough. Among that advice, the authors advocate that you get out of your comfort zone and try something you've been meaning to, but hadn't gotten around to. My comfort zone being C# and ASP.NET at that point, I decided to try Node.js, an unmistakably UNIX-y platform for JavaScript development on the server-side, certainly a break from my Microsoft-ridden development experience thus far.

I learned a ton from that experiment, and [ended up with this blog][first] 🦄  where I would write about everything I learned in the process. Around half a year later I got an idea where I'd put my years of experience in C# design into a book about JavaScript. I contacted Manning and they jumped at the opportunity, helping me brainstorm and turn raw ideas into something more deliberate and concise.

[![Pony Foo. Humble beginnings. The design was terrible, but it got the job done!][pfold]][first]

When it comes to this website, I [iterated it into an over-engineered pile of code][overengineer] that does everything but brew coffee. It's definitely not Ultima Online, but part of Ultima's magic was in the learning process, and I still get to implement thrilling stuff once in a while, such as the [`git` integration][git] or the [Pony Foo Weekly][pfw] back-end.

You know the rest: I worked for a few startups, wrote some more, did some consulting, and now love every day at [Elastic][elastic] using ES6, Webpack, React, Redux, and friends.

*It **still** feels like a game to me.* What's your story?

[ruo]: https://github.com/runuo/runuo "runuo/runuo on GitHub"
[pp]: http://amzn.to/2d1qGg4 "The Pragmatic Programmer: From Journeyman to Master (Hunt, Thomas – 1999) is a timeless classic you should seriously consider reading"
[oss]: https://web.archive.org/web/20110721010354/http://www.runuo.com/community/threads/runuo-2-0-rc1-duel-pit-system.74599/
[sw]: https://github.com/runuo/runuo/tree/d038571b412188c2f4416f29846f11275c1e7bcb/Scripts/Spells/Spellweaving "Spellweaving code for RunUO on GitHub"
[first]: /articles/pony-foo-begins "Pony Foo Begins"
[elastic]: https://www.elastic.co/ "Elastic is the company behind Elasticsearch"
[uopic]: https://i.imgur.com/wmv3GRL.jpg
[pfold]: https://i.imgur.com/5aOGBKJ.png
[overengineer]: /articles/most-over-engineered-blog-ever "How Pony Foo is ridiculously over-engineered — and why that is awesome"
[git]: /articles/two-way-synchronization-for-a-web-app-and-git "Two-way Synchronization for a Web App and Git on Pony Foo"
[pfw]: /weekly "A newsletter about the open web, highlighting the most important news about the web every thursday"
