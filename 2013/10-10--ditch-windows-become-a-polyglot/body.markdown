![angelfire.png][1]

In retrospect, no one visited it either, being completely honest to ourselves. Eventually my understanding of the web evolved, and I started reading about PHP on my spare time. You mean I can let people _(well, just myself again, really)_ **log in to my site**?? Just like the big names? Whoa. And so we learned PHP, until one day I became tired of that nonsense, and I decided I was going to write games. Turned out designing games in VB6 wasn't very reasonable either, and `On Error Resume Next` didn't seem to be helping me out a lot. I managed, anyways. I read about game development, followed tutorias, and soon enough I was setting up my first game loop, complete with a menu screen and a 2D tilemap, as well as a few frames worth of characters. I clearly didn't have what it took to design these graphics, though, but sketching them in paint _seemed to suffice_, at that time.

> Looking back, I _immensely regret not being more careful about backing up my code as a child_, I would have all this awesome code that never saw the light of day. Only had I learnt about version control sooner, but that was the price of the few kids around who loved to spend his time typing things into a keyboard.

### Windows, windows, windows

I never really paid any attention to _Linux_ as a kid, and I definitely couldn't afford a Mac at the time. I had heard about LInux, but I was under the impression that the terminal was all there was to Linux. I didn't want to go back to a DOS-like interface, **DOS sucked**. So I was stuck with Windows, and programming in PHP, I didn't know any better.

![cat-stuck-in-window.jpg][2]

Later on, I became friends with the owners of a game I played on a daily basis (the game server was a fork of an open source project called [RunUO](http://www.runuo.com/ "RunUO Ultima Online Emulator")). Almost as a matter of fate, they explained that they had a falling out with the programmer they had, and asked if I could take a look at one of their scripts, because they had been meaning to put together a quest, but they had no idea what they were doing. I didn't even know about **C#**, but I gave it a shot. Over time, I ended up becoming a contributor to the project, and at one point, I even learned that people could, _you know..._, **get paid** for coding in ***C#***.

I started working with the language I dearly loved, and I couldn't have been happier. I worked with **C#** for years, and I had spent a few years on my own working on the RunUO project. As I worked, I moved closer and closer to the web I loved so much when I was younger, but this time I came prepared, it wasn't merely [FrontPage](http://en.wikipedia.org/wiki/Microsoft_FrontPage "Microsoft FrontPage on Wikipedia") anymore, **C#** was the real deal!

Still, the command-line was a complete stranger to me. `cmd`, which comes bundled with Windows, is **absolutely god-awfully useless** if you want to get anything done solely typing into it. When I actually had to start using a terminal (not by choice, but simply because there weren't any good graphical `git` interfaces), I started out with `bash`, and used that for a while before moving to [PowerShell](http://en.wikipedia.org/wiki/Windows_PowerShell "Windows PowerShell on Wikipedia"). I was so repelled by the command-line, that I thoroughly enjoyed moving my mouse around, right clicking on things, and picking options out of a menu.

**C#** _taught me a lot_ about architecture, and application design in general. However, as the architecture fanatic I ended up becoming, _my heart was always with the web_. I started moving towards the front-end, and preoccupying myself with the web matters again. Little did **C#** do for me to become an avid terminal user, but that was about to change.

### A New Order

Around a year ago, I bought a variety of books ([reviewed in this article](/2013/05/21/recommended-reading "Recommended Reading")) on UX, architecture, and best practices, [one of those books](http://www.amazon.com/dp/020161622X "The Pragmatic Programmer") _completely changed me_ as a developer. Among other valuable insights, the authors prompted me to get out of my comfort zone, and learn a new language. I picked **Node.js**, even though I had no idea what it was, i sure _sounded_ interesting. I decided to start working on something with it right away, and this blog [became my pet project](/2012/12/25/pony-foo-begins "Pony Foo Begins").

Learning Node has been a wonderful experience for me and it has now become _part of my daily work_, much like C# did before. A tremendous difference, however, is that Node isn't _biased towards Windows_ at all, which forced me to actually learn what happens under the covers when you compare it against the abstractions C# does for us. These abstractions, even though powerful, end up obscuring fundamental aspects of our programs, and hindering the sustained learning of those of us with a thirst for knowledge.

> When I first started working with Node, still the Windows-infected user that I was, I dreaded the idea of typing into the command-line. I already was a PowerShell _(PS)_ user by then, but using it for dealing with `git` was enough, _thank you_. Setting up PS is unfortunately pretty much as good as it gets on Windows, and yet a complete nightmare for any Windows user, completely unaware of what the command-line even is. Why do I have to type `node app` into my command-line?

As time went by, and I started using more and more `npm` modules written by complete strangers, the open-source community started becoming a whole different level of appealing to me, I _needed_ to get involved. Yet, a lot of the packages I was trying to use weren't careful enough to conform with Windows' upside-down OS design, and I started becoming disenchanted about Windows, even though I had recently acquired an _early RTM version_ of Windows 8, and loving it, if only that stupid command-line behaved...

Then I got a job working on Node, even though all I had to show for was this blog, and my hand was forced. I just had to pick up my old, _2008 era_ **MacBook Pro** and work there, rather than my PC. The reason was, _simply enough_, I just couldn't get the application to work on Windows. I moved on, and soon afterwards it start becoming apparent what I had been missing. I started using the terminal more heavily than I had ever done on Windows, and I was loving the idea of being able to do anything, without _having to download yet another program for it_ (that's the important part, all of this came bundled with it, I didn't have to download a thing, it was just another part of the OS). Before I even realized it, I was `curl`ing, `grep`ing, `sed`ing, and piping my way around the command line greatly improving my productivity. I thoroughly enjoyed installing packages right from the command-line with `npm`, as opposed to having to click through things with `NuGet` in the case of **C#**, dealing with _unreasonably case-sensitive_ commands in PS, and barely even knowing about alternatives to compile than pressing <kbd>F6</kbd> on the **Visual Studio IDE**.

> The difference _between Windows and everyone else_, is that you won't really learn much about development, _other than what Microsoft imposes on you_, and even if you had that craving of learning just about anything, you wouldn't be aware about this constraint. I _sure as hell wasn't_.

I'm not saying that I've learnt it all, _**extremely** far from it_, but ditching Windows _opened my eyes_ to an endless world of possibilities I didn't even know existed before. Besides, being able to pass environment variables to processes like `PORT=3000 node app` in your terminal is too good to pass up.

A couple of days ago, I decided to install [Ubuntu](http://www.ubuntu.com/ "Ubuntu Linux") on my PC. One of the first things I did was create a file where I'd place all the steps taken to set up my environment, and eventually I created [a repository on GitHub](https://github.com/bevacqua/dotfiles "My dotfiles on GitHub") so i wouldn't have to repeat myself when installing Ubuntu again. This has never been a_ feasible option_ for Windows, at least not for me.

It _might've taken me 25 years_, but **you can be damn sure the change was worth it**.

  [1]: https://i.imgur.com/6ApllqK.png "Precious, free hosting! Where do I sign?"
  [2]: https://i.imgur.com/HJsAINu.jpg "Halp!"
