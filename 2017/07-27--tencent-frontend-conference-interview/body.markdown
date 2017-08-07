::: .c-dark-turquoise
*Interviewer:* **What does Argentina’s tech community look like?**
:::

It's in a much better shape than it was, say, 5 years ago. Nowadays, we have JSConf, CSSConf, NodeConf, and the smaller meetups people are also used to like BeerJS, NodeSchool, and NodeBots. It’s definitely growing. NodeConf started last year and this year [it's shaping up to be fantastic][nodeconf]. With conferences and meetups on a regular schedule, people can engage more as a community, and not just attend some event once and then completely forget about it.

::: .c-dark-turquoise
*Interviewer:* **What attracts you in JavaScript?**
:::

We have a fantastic developer community. We also have a wide assortment of frameworks and libraries to pick from, and while some may dub that JavaScript fatigue, I think it's crucial to innovation and the advancement of the web platform. When React came out in 2013 it was massively misunderstood and mocked by the community as a fad, with criticism against Facebook for trying to reinvent the web. Now React is almost ubiquitous and many can't imagine a situation where they wouldn't use it. Such a paradigm shift would be a lot harder to come across in a community where we didn't stop and think how the ecosystem can improve.

::: .c-dark-turquoise
*Interviewer:* **There are lots of frameworks in JavaScript. How can we keep up with the constant change?**
:::

There’s probably a framework coming out right as we speak. The important thing is to avoid the temptation of jumping onto the latest and shiniest bandwagon, whatever that may be. It's more about figuring out the trends, what’s useful and helpful for me and my projects.

If you started a project 2 years ago and it's "still" using Angular, there might very well be no need to migrate to React. The fact Angular is a little bit lower than React is often negligible especially when considering the amount of time you'd put into a migration like this for any moderately large application. It always depends on your requirements. The consistency of using the same tool for a while is valuable.

It's important to not sink our heads in the sand for too long, though. This is why I insist on keeping things in your radar, so that if you take on a new position you can hit the ground running without spending a month sifting through documentation for an unfamiliar framework. It's about finding the right balance: get a solid footing on the fundamentals of JavaScript or whatever else you want to learn, and stay informed about developments in the community. Try things out as needed, but don't feel an obligation to do so. Be deliberate in not jumping into everything just because it is new.

::: .c-dark-turquoise
*Interviewer:* **Could you give us some advice on learning progression for JavaScript?**
:::

It’s really important to figure out how you learn things best. Some people love to watch screencasts or other videos. Some people enjoy a good audiobook. Some people like to read digital books. Some prefer to read physical books. I’m really bad at learning through vision. I need to read. If I watched a video and I need to understand it well, I'll spend a lot longer than if I just read equivalent content on the same topic. What I’m trying to say is that you should figure out whether you are a visual learner or if you prefer written content. Maybe you prefer to learn as you go and by trying things out and breaking them apart, or perhaps you prefer something more structured like a course with a series of screencasts or long-form articles *--- or formal education, if that's your cup of tea.*

Once you're clear on that point, you can start by understand the core principles of writing JavaScript: syntax, grammar, algorithms, data structures, things like hoisting and scoping, and so on. Once you have a solid foundation, you could move into ES6 and learn modern features. At the same time, you might focus on a single framework, Angular or React, but pick one at first and learn it well, try to avoid jumping from topic to topic without learning anything substantial about it. You could check out the documentation or browse source code to figure out how it works.

I implemented my own frameworks to learn how similar frameworks work under the hood. I did this with many libraries as well. I think it’s an effective way to experiment and come up with your own understanding of how things work. I would recommend [12factor.net][12f]. It lays out 12 different principles for robust application design in terms of security, configuration, scalability, etc. I found it invaluable when I was first getting into open-source. I'm also a huge fan of [The Pragmatic Programmer][pp], far and away the best book I read on our craft.

::: .c-dark-turquoise
*Interviewer:* **Currently, you are writing the Modular JavaScript book series. Why do you focus on modular JavaScript?**
:::

In the early days, JavaScript was an after-thought at best. We copy-pasted all the things: comment systems, widgets, and so on. As the language and our understanding of the web evolved, this ceased to be the case. Large JavaScript applications introduced the need for more robust separation of concerns and proper architecture design in the client, just as we have been doing with server-side languages. Nowadays, JavaScript applications have even more logic in them, and hence, modules.

In [my book series][books], I’m teaching how to write concise and single-purpose modules. The reason is that we need modules to be specialized so that they can reused, tested, and document in isolation. Scalability is an important aspect of this, in terms of architecture. When you have 200 different modules with 5,000 lines of code each, it can be mind-boggling to work with them and it's surprising if you get anything done. In contrast, if you have 5,000 modules comprised of 200 lines of code, it's a lot simpler to understand how pieces of code are related to each other, and what they do.

The series discusses language features in and after ES6, how to design and maintain highly modular applications, how to test our modules, and effectively deploy these applications.

[nodeconf]: https://2017.nodeconf.com.ar "NodeConf Argentina 2017"
[12f]: https://12factor.net
[pp]: http://amzn.to/2uC0BhU
[books]: /books
