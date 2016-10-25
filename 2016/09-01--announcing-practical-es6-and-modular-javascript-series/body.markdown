# The Modular JavaScript Book Series 📦

Modern application design is hard.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wgsExttW9BY" frameborder="0" allowfullscreen></iframe>

Modular JavaScript is an open effort to improve our collective understanding of writing robust, well-tested and modular applications. It consists of five books, each of which explores a key aspect of JavaScript development — comprehensively. The books are produced in the open: anyone can track their progress, report issues and contribute fixes or content. A free-to-read version is available online! Digital and print books can be purchased via O'Reilly Media.

<iframe width="560" height="315" src="https://www.youtube.com/embed/Ky0ZSQW0lBg" frameborder="0" allowfullscreen></iframe>

This may sound surprising if you've never written a book, but the Table of Contents is something you draft before you even start writing the first page, and it typically undergoes several radical changes as the book is being developed. It could be that the content needs better flow, some topics are deemed irrelevant or out of a particular scope, or different topics that need to be introduced.

I've decided to make Modular JavaScript an open effort, which is something I'll circle back to after describing each book in the series. First off, check out the description for each book, and their currently proposed table of contents.  
I invite you to look them over and chime in with your opinions. The first book, Practical ES6, is unlikely to change a lot. The rest of them are quite likely to change dramatically as they're developed.

# Book --- Practical ES6

::: .mde-right.mde-pad-20.mde-33.mde-text-center
[<img src='https://i.imgur.com/DKya6OE.png' alt='Book cover for Practical ES6' />][toc]
<br>
<sub><em>Boils down the essentials of ES6 with plain terminology and easy-to-follow code examples.</em></sub>
:::

The first book, *already in development*, is dedicated to ES6. When it comes to books on a language, these tend to be more of a reference style but not so much practical. **Practical ES6** aims to provide practical instances in which you could leverage the different language features, at the same time as it teaches you how each feature works.

Code examples are meant to be readily consumable and applicable to real-world JavaScript applications. This is something I strived toward in the JavaScript Application Design book, although the code samples were sometimes a bit more contrived than they had to be.

A friendly tone paired with wording that's light on jargon are aimed towards making the reading experience as pleasant as possible.

This book will go over each feature in ES6 in detail, including how to use Babel and friends, as well as practical advice on what works and what doesn't.

<div class='mde-clearfix'></div>

<details>
<summary>Table of Contents</summary>
<h3 id='toc-practical-es6'>I. Practical ES6</h3>

1 Introduction to ECMAScript 6  
1.1 What is ES6  
1.2 First Look at ES6

2 Build Tooling Around ES6  
2.3 Introduction to Babel, webpack, browserify, eslint  
2.4 Compilation with Babel

3 ES6 Essentials  
3.1 Object Literals  
3.2 Arrow Functions  
3.3 Assignment Destructuring  
3.4 Rest Parameters and Spread Operator  
3.5 Template Literals  
3.6 Let and Const Statements

4 Classes, Symbols, and Objects  
4.1 Classes  
4.2 Symbols  
4.3 Improvements to Object

5 Iteration and Flow Control  
5.1 Promises  
5.2 Iterators  
5.3 Generators

6 Collections  
6.1 Sets  
6.2 WeakSets  
6.3 Maps  
6.4 WeakMaps

7 Proxies

8 Built-in Improvements  
8.1 Number  
8.2 Math  
8.3 Strings and Unicode  
8.4 Array Methods

9 JavaScript Modules  
9.1 CommonJS  
9.2 Modules in ES6  
9.3 Exports and Imports  
9.4 Module Loading  
9.5 Interoperability

10 Practical Considerations
</details>

::: .mde-text-right
# Book --- Mastering Modular JavaScript
:::

::: .mde-left.mde-pad-20.mde-33.mde-text-center
[<img src='https://i.imgur.com/wOz6fTX.png' alt='Book cover for Mastering Modular JavaScript' />][toc]
<br>
<sub><em>Managing complexity has never been so straightforward.</em></sub>
:::

The second book in the series gets more interesting. Once we've established that the reader knows ES6, then we can take it for granted. That's one of the amazing aspects of writing a book series, you can get away with things that wouldn't make a ton of sense in standalone books.

This book is much more theory-intensive than the first, as it involves design patterns, guidelines for "thinking in modules", principles, what consistitutes a great module, etc. That said, the book will probably involve developing and publishing a number of modules exclusively for the purposes of learning, so you can expect a decent amount of code here as well.

The modular nature of the series allows me to dig deep in this book and include tips on publishing to `npm`, versioning, writing great documentation, and everything else that makes a perfect package.

<div class='mde-clearfix'></div>

<details>
<summary>Table of Contents</summary>
<h3 id='toc-mastering-modular-javascript'>II. Mastering Modular JavaScript</h3>

1 Module Thinking  
1.1 Introduction to Module Thinking  
1.2 Brief History of Modularity  
1.3 Benefits of Modular Design  
1.4 What Constitutes a Module?  
1.5 Why Modular JavaScript?  
1.6 Future of JavaScript

2 Modularity Principles  
2.1 Motivations  
2.2 Single Responsibility Principle  
2.3 Small and Simple  
2.4 Portability  
2.5 Composability

3 Shaping a Module  
3.1 API First  
3.2 Internal Design  
3.3 Loose Coupling  
3.4 Semantic Versioning  
3.5 Documentation  
3.6 Testing

4 Modular Patterns and Practices  
4.1 Revealing Module  
4.2 Object Factory  
4.3 Composition Over Inheritance  
4.4 Events  
4.5 Conventions

5 Developing Modules  
5.1 Leveraging `npm`  
5.2 Debugging Dependencies  
5.3 Introducing Webpack  
5.4 Source Mapping  
5.5 Automated Publishing

<sub>This table of contents hasn't been fleshed out yet. We're open to your suggestions! ⚡</sub>
</details>

# Book --- Universal JavaScript, Modules Everywhere

::: .mde-right.mde-pad-20.mde-33.mde-text-center
[<img src='https://i.imgur.com/F6cRf0F.png' alt='Book cover for Universal JavaScript' />][toc]
<br>
<sub><em>Learn the essentials of code reuse and plumbing -- even across platforms.</em></sub>
:::

This book covers how we can create entire modular applications. We'll be looking at how to reuse code across a codebase when it comes to shared logic between the front-end and the back-end.

Module interaction design will play a big part here, where we'll try and come up with a plan to develop modules that can be plumbed together with minimal friction.

The book sets out to answer the question: _"How do we scale out a module approach to fully flesh out an application without ending up with 500+ LOC modules?"_

<div class='mde-clearfix'></div>

<details>
<summary>Table of Contents</summary>
<h3 id='toc-universal-javascript-modules-everywhere'>III. Universal JavaScript, Modules Everywhere</h3>

1 Universal JavaScript  
1.1 Reasoning  
1.2 Back to Browserify  
1.3 The “browser” Field  
1.4 Considerations  
1.5 Authoritative Examples  
1.6 A Case Study

2 Interconnecting Modules  
2.1 Simplicity vs. Complexity  
2.2 Module Interaction Design  
2.3 Scaling Out  
2.4 Documenting and Testing

<sub>This table of contents hasn't been fleshed out yet. We're open to your suggestions! ⚡</sub>
</details>

::: .mde-text-right
# Book --- Testing JavaScript Modules
:::

::: .mde-left.mde-pad-20.mde-33.mde-text-center
<img src='https://i.imgur.com/mrK7DSP.png' alt='Book cover for Testing JavaScript Modules' />
<br>
<sub><em>Built on the foundation of testable code laid out in previous books in the series. Plenty of real life examples!</em></sub>
:::

There is so much ground to cover in testing, that it deserves a book of its own in the series. We'll start by covering the basics, such as linting, and then move onto unit testing. The beauty of the modular nature of the series strikes again, where we can explore unit testing after we covered how modules should be designed in order to maximize testability in previous books.

Automated browser testing, asynchronous tests, integration tests, and every relevant kind of testing you can think of will probably be included in this book.

<div class='mde-clearfix'></div>

Continuous testing will be covered as well, and it will in fact be an excellent segue into the next and last book of the series.

<details>
<summary>Table of Contents</summary>
<h3 id='toc-testing-javascript-modules'>IV. Testing JavaScript Modules</h3>

1 Module Testing  
1.1 Linting  
1.2 Overlooking Implementations  
1.3 Testing Your First ES6 Module  
1.4 Assertions  
1.5 Spies and Stubs  
1.6 Mocking Modules

2 Integration and Continuous Testing  
2.1 Integration Testing  
2.2 Visual Testing  
2.3 Testing Locally  
2.4 Continuous Integration  
2.5 Headless Browsers  
2.6 Zuul and SauceLabs

<sub>This table of contents hasn't been fleshed out yet. We're open to your suggestions! ⚡</sub>
</details>

# Book --- Deploying Modern JavaScript Applications

::: .mde-right.mde-pad-20.mde-33.mde-text-center
<img src='https://i.imgur.com/YiJzLUB.png' alt='Book cover for Deploying Modern JavaScript Applications' />
<br>
<sub><em>Details on how to set up hassle-free deployment processes that don't get in the way of productive development, yet are optimized for performance.</em></sub>
:::

Optimizing an application for deployment is not exactly something you want to leave for last. After all, that's the state in which all your users will be consuming your application. The series will progressively build towards something that's easily optimized and then deployed.

This book covers different ways you can deploy your JavaScript applications, such as via containers or PaaS solutions. You'll learn how to better optimize for bundle size, HTTP/2, and the trade-offs each optimization implies.

<div class='mde-clearfix'></div>

<details>
<summary>Table of Contents</summary>
<h3 id='toc-deploying-modern-javascript-apps'>V. Deploying Modern JavaScript Applications</h3>

1 Deploying Modular Applications  
1.1 Compilation  
1.2 Optimization  
1.3 Universal Execution  
1.4 HTTP/2 and the Road Ahead

<sub>This table of contents hasn't been fleshed out yet. We're open to your suggestions! ⚡</sub>
</details>

# An Open Effort for Modular JavaScript

As I mentioned earlier, Modular JavaScript will be an open effort. What does this entail? Well, that involves a four-part answer. *Let's start with the free stuff!*

## Practical ES6 is *Free* to Read!

The book is publicly available in HTML format and **free forever**. Each book chapter is styled similarly to how Pony Foo blog posts _— such as the one you're reading —_ are styled, which makes for a fairly enjoyable read as far as HTML books go.

*Every book in the series will be distributed in this way.*

::: .mde-right.mde-pad-20.mde-50.mde-text-center
[<img src='https://i.imgur.com/F41tiVr.png' alt='The online version of Practical ES6' />][toc]
<br>
*<sub>Chapters are easy on your eyes and every section title comes with its own permalink.</sub>*
:::

It took me a bit of time but I've managed to get the `git` repository to trigger builds on the O'Reilly build server. The build server then pings back to ponyfoo.com, letting the site know a build is ready. Lastly, Pony Foo finally downloads the updated HTML files for the web version. Luckily I developed the code in such a way that I will be able to share HTML versions of other books in the series effortlessly.

Taking a page from Buffer's **radical transparency** program, where they advertise salaries, equity grants, revenue, and other figures of interest, I will do my best to make Modular JavaScript's numbers available on a regular basis. I'll disclose the amount of units sold, my losses and earnings, and how much money I end up generating through this adventure.

> The free-to-read version of Practical ES6 is subject to [the same license][license] as the rest of the content I publish on Pony Foo: [*Creative Commons Attribution-NonCommercial-ShareAlike*][license].

_**Read** the [HTML version of the book][toc] on Pony Foo! 🦄_

## Source Code Repository!

::: .mde-right.mde-pad-20.mde-50.mde-text-center
[<img src='https://i.imgur.com/RjdWVDK.png' alt='GitHub repository for Practical ES6' />][contrib]
<br>
*<sub>All chapters, source code, and images!</sub>*
:::

I want this book series to be as widely available as possible, and the best course of action for that purpose was to release the vast majority of its contents to the open-source community.

The book chapters, code samples, and related graphics are all open-source and free to read online. The repository is the same one I work on while writing the book. You can help me in real-time, or just take a peek at my writing process and progress.

O'Reilly provides their authors with `git` repositories under a system dubbed "Atlas" _— the same system that handles the HTML and PDF build jobs._ They also offer a way of using a GitHub repository while keeping their remote up to date via git hooks.

> The GitHub repository is automatically synchronized with the website using webhooks and the O'Reilly Atlas API.

Those are the implementation details, but it means I can offer an open-source repository for the books _— the same repository I will be working on myself._ This means I can take issues, pull requests, and everything directly on GitHub. You can fork the book, fix some typos or add a new paragraph, and submit a PR.

When I merge a PR, the website will be updated after an automated build _courtesy of the O'Reilly Atlas service!_

_**Contribute** to the [source code repository][contrib] on GitHub! 👏_

::: .mde-text-right
## A Crowdfunding Campaign!
:::

::: .mde-left.mde-pad-20.mde-66.mde-text-center
[<img src='https://i.imgur.com/02Cqf9W.png' alt='Crowdfunding campaign for Modular JavaScript' />][campaign]
<br>
<sub><em>Every contribution counts! Help me spread the word?</em></sub>
:::

Offering all of this content free of charge is amazing because I can ensure that anybody who's interested in JavaScript can learn more about it. The satisfaction alone doesn't pay any bills, though.

At the same time, I have to manage to write the series somewhere in between my day job at **Elastic** _(it's the absolute best company — [we're hiring!][hire] 🦄💖🔎🎉)_ and my <del>night job</del> <ins>life bliss</ins> [being a husband][married].

That's why I'm asking for your help with a crowdfunding campaign. The main concern in the campaign is to keep me motivated to find the time to see the series to its end.

<div class='mde-clearfix'></div>

The first book is already well underway, and I'd love to be able to justify the dedication that each book in the series deserves.

There's quite a few perks in the campaign -- as is the norm with crowdfunding these days. I'm sure you'll find something to your liking. Go [check out the campaign][campaign] and let's make this happen!

<iframe width="560" height="315" src="https://www.youtube.com/embed/MpP4MHFrIF4" frameborder="0" allowfullscreen></iframe>

_**Participate** in the [crowdfunding campaign][campaign] on Indiegogo! 💸_

## O'Reilly Media Early Release!

::: .mde-right.mde-pad-20.mde-33.mde-text-center
[<img src='https://i.imgur.com/DKya6OE.png' alt='Book cover for Practical ES6' />][er]
<br>
*<sub>O'Reilly Media is pretty amazing. I have over a dozen of their books on my shelves!</sub>*
:::

I've partnered with O'Reilly Media to publish the book. This is a paid offering that includes a PDF ebook and eventually a print book. The published book is a great way to show your support for my work, by paying a bit for it _— and telling your friends how awesome my writing is._ 😘

We'll start with an Early Release, where you will get the first few chapters in ebook format. As new chapters come out and old ones get improvements, you'll receive those updates at no extra cost to you. You'll also get an opportunity to steer the direction of my writing efforts by reporting errors and delivering book reviews.

This is a great way to stay in touch with me through the writing process and letting me have it when drafts are not up to your expectations, so that we can improve the book iteratively before it goes to print.

_**Purchase** the [Early Release][er] from O'Reilly! 📓_

<div class='mde-clearfix'></div>

## Screencasts? *Probably!*

There's a lot of ways to present content, but screencasts are an ideal companion for a deep-diving book series. The idea would be to interleave some content discussed in the book with an even more practical focus -- perhaps showcasing concepts using code samples. Screencasts would be produced separately after each book goes to production *(an intensive copyediting and proofing process publishers undergo before a book goes to print)*.

I'd love to shoot screencasts to complement the books. *If the [crowdfunding campaign][campaign] goes well, we could definitely make this a reality!*

# Thanks! 👋

Thanks for reading this *needlessly long announcement* about Practical ES6 and the Modular JavaScript Book Series, and please consider sharing this piece so that more of our peers can check out [the web version of Practical ES6][toc] and maybe even help me out with the [crowdfunding campaign][campaign].

# Where To?

> 💳 **Participate** in the [crowdfunding campaign][campaign] on *Indiegogo*  
> 🐤 **Share** a [message on *Twitter*][tweet] or within your social circles  
> 🌩 **Amplify** the [announcement on social media][clap] via *Thunderclap*  
> 📓 **Purchase** the [Early Release][er] from *O'Reilly*  
> 👏 **Contribute** to the [source code repository][contrib] on *GitHub*  
> 🦄 **Read** the online [HTML version of the book][toc] on *Pony Foo*  

Talk soon. 🐗

[hire]: mailto:nico@elastic.co "Get in touch with a cover letter and your resume!"
[married]: /articles/just-married "Just Married! announcement on Pony Foo"
[license]: https://ponyfoo.com/license "Licensing Terms on Pony Foo"
[repo]: /s/practical-es6-repo "mjavascript/practical-es6 on GitHub"
[clap]: /s/modular-javascript-thunderclap "Back the Thunderclap campaign!"
[tweet]: /s/modular-javascript-tweet "Send out a tweet promoting the Modular JavaScript launch"
[toc]: /s/practical-es6-read "Practical ES6: A Practical Dive into ES6 and Maintainable JavaScript Modules"
[contrib]: /s/practical-es6-repo-contrib "mjavascript/practical-es6 on GitHub"
[campaign]: /s/modular-javascript-indiegogo "Indiegogo campaign for Modular JavaScript: A Pragmatic JS Book Series"
[er]: /s/practical-es6-early-release "Modular JavaScript: Practical ES6"
