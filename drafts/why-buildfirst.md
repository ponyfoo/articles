# Why You Should Build First

_Build First_ is a set of principles I've collected over the years and which is influenced by several sources. This article aims to explain the driving principles behind the [JavaScript Application Design][1] book, the authors and resources which influenced its creation, and detail the benefits of taking _Build First_ for a spin.

In this article, my aim is to help you discover the _Build First_ paradigm, its applications, strengths, and pitfalls.

...

[1]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"





# Influence

[The Lean Startup][1] has already become a classic, taking the Toyota Production System lean practices to the software development world. These practices [can be summed up][2] in what Ries calls the build-measure-learn feedback loop. Rather than building out an application and then _hoping_ for the best, Eric prompts us to collect feedback from our humans, using the [Build-Measure-Learn][2] feedback loop, and then decide which features are best _for them_. Following this _iterative approach_ to development is similar to Agile, but with the difference that any feature should be immediately releasable. Kanban is one way to approach lean development, and it has a number of benefits over traditional Agile.

- Deployments are made easier by deploying more often, maybe even several times a day
- Feedback on which features customers actually engage with gets to us faster
- Minor fixes don't have to wait until a release day a week from now, because those happen whenever we need one
- Hot fixes can be pushes through the ranks immediately
- Recently introduced bugs can be quickly identified because features are released in short bursts, rather than commiting huge change lists in each individual release

Build First is also influenced by the [12 Factor App Manifest][3], written by one of Heroku's co-founders. In this document, Wiggins details their approach to application architecture, configuration, scaling, and deployment. **This is a read I could not recommend more often.** Granted, working in Node.js and open-source projects had already taught me a lot about the topic, but it helped me further advance my treatment of secure configuration and scalable application design. Below is one of the subtle (yet, _key_) take-aways I came up with, after working in the open-source JavaScript community for a while, reading and following Heroku's _12 Factor App_ manifest.

> Write closed source projects as if they might be open-sourced overnight

That is, don't assume the closed-source nature of your project to be _secure enough_ for you to place sensitive data such as API credentials or email authentication information directly in the project. Instead, keep private data in environment variables, or use encryption if you want to keep it in the repository safely. This won't only help you write code that's safer, but it'll also make the code easier to scale, too!

There's also the [Pragmatic Programmer][4], this is a book I've [recommended][5] in [the past][6], and which leads all of my software development book lists. The authors encourage us to think about the code we write, to develop a mindset where we're able to design orthogonal code which stays **maintainable over time**, and many other useful pieces of advice.

> If you read a single programming book in your lifetime, make that the [Pragmatic Programmer][4].

One of my favorite parts of the book talks about **broken windows**. It explains how seemingly _innocent, quick fixes_, when left unrepaired, can spell the demise of a code-base. It explains the phenomenon comparing it to a building with a broken window. If that window isn't repaired, then other windows will get smashed, maybe a car's window. Abandoned cars and buildings start setting a different environment in the neighborhood. Before long, theft and pillaging start making appearances, and soon the neighborhood becomes impossible to live in.

Armed with these teachings, _Build First_ is a mixture between solid application design principles and the added twist of putting together a build process right off the bat, **rather than waiting for a disaster to happen**.

# Driving Principles


...


- automate everything, right off the bat
- less headaches, leaner process
tighten your feedback loop
better performance (min concat)
more productive (sprites, less gruntwork)
- automated testing is super easy
-deployments in a single step
- database rollbacks, you name it!
- it's easier with Grunt

[1]: http://www.amazon.com/dp/0307887898 "The Lean Startup book, by Eric Ries"
[2]: /2013/07/29/lean-development-principles "Lean Development Principles"
[3]: http://12factor.net "Heroku's 12 Factor App manifest"
[4]: http://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X "Find it in Amazon"
[5]: /2014-01-01/a-year-in-review "A Year in Review"
[6]: /2013/05/21/recommended-reading "Recommended Reading"
