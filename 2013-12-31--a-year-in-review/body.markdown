Everything being open-source really helped my learning process, and I quickly learned best practices. Right off the bat I used Heroku as my hosting environment, and I found myself doing lots of the things proposed by [12factor][1], before discovering that document.

## Asset management, and `assetify`

Soon afterwards I made my first contribution to open-source, in [assetify][2], an asset manager that packages up bundling, minification, and cache busting in a piece of middleware. At the time I didn't yet have the love I've been garnering as of late for applications that strive to be as static as possible, and assetify was [_kind of very tightly coupled_][3]. 

A few weeks later, I [came across Grunt][4], and wrote [a plugin to match assetify][5], so that the build step could be automated. I kind of fell in love with Grunt, and I started using it for everything, writing lots of little plugins to use with it.

Later on, I would split the functionality in half, allowing consumers to run the build in a separate process from what was ultimately serving the assets. However, I wasn't really using assetify anymore, except for this blog, which I rarely worked on anymore.

> In retrospect, one of my biggest mistakes at this stage was attempting to use Grunt to not only build the application, but run it, too. 

## Early Blogging Experience

At the beginning, I wrote blog posts about pretty much anything that came to my mind. To get things going, I figured I'd start by talking about what I was working on in the blogging engine, my only real work using Node, at the time. As such, I wrote focused posts that explained some of the considerations I had to make while building the blog.

I wrote an [introduction to SEO and Content Indexing][6] in AJAX applications, which explained the steps I took to enable Google to crawl my site, while keeping the <del>awesome</del> **extremely irritating** AJAX functionality in place. I say _extremely irritating_ because I made the mistake of not [rendering the content server-side][7] during the initial page load, which proved to be a huge hit to the usability of the blog, and _something I intend to correct at some point in 2014_.

I also wrote small blog posts where I described what I had learned through usability books at the time, such as [Defensive Design][8], a book that can be summarized in designing consistent error messages and interfaces in general, and [The Design of Everyday Things][9]. In retrospect, I think these blog posts could've been a lot more detailed than they resulted.

> Next year, I'll make it a thing to give each article a few days before hitting the <kbd>Post</kbd> button.

## My Own MVC Framework

That being said, I was particularly happy with how the code looked like, I didn't use any libraries other than jQuery _(which I regret doing)_, but keep in mind that at that time I _hadn't even tried out Angular_ yet. My favorite part was how I made it so that if a response came with a particular HTTP status code, my little framework knew what to do with that. It'd prepare the pertinent validation message view, and place that in the context of where the request originated from.

![400.png][10]

If there wasn't a context, then the validation message would pop up in a dialog, [amusing stuff][11]! Totally irrelevant in a blog, of course. Developing [my own MVC framework][12] was _thrilling_, nonetheless. Regrettably, I didn't write it in isolation, otherwise I would've continued working on it, maybe when I rewrite the engine I'll put it to good use again. It had wonderful little things, you'd code up views, bind them to routes, and request that a given view fetches some data from the server before getting rendered.

Of course, the views were rendered using the [Mustache][13] engine, but some things, like [HTTP requests getting tracked][14], because I wanted to write a caching layer in the client-side, as to fix the usability a bit without the need for server-side rendering, but never got to it. The `thin` layer in **NBrut** did enable things like status code validation, and aborting requests when they weren't no longer necessary (for instance, when fetching an article and then promptly clicking on the header, getting back to the home page).

Actually using the framework [wasn't as bad as you might think][15], but the thorough lack of documentation makes it really hard to follow the code around, even for me, and I wrote the thing a few months ago.

> In the future, I'll abstain from tightly coupling major pieces of functionality together, and instead force myself to write reusable components which are well documented.

## Reading, Reading, Reading

Late in 2012 I bought a bunch of software related books, and I've read a large number of them. Here are the ones I read. Can you spot the one that's not so much about software?

- [Test-Driven JavaScript Development](http://www.amazon.com/dp/0321683919 "Check it out on Amazon") _(ebook)_
- [Responsive Web Design](http://www.abookapart.com/products/responsive-web-design "Check it out on AListApart") _(ebook)_
- [Mobile First](http://www.abookapart.com/products/mobile-first "Check it out on AListApart") _(ebook)_
- [Lean UX](http://www.amazon.com/dp/1449311652 "Check it out on Amazon") _(ebook)_
- [Testable JavaScript](http://www.amazon.com/dp/1449323391 "Check it out on Amazon") _(ebook)_
- [Programming Pearls](http://www.amazon.com/gp/product/0201657880 "Check it out on Amazon")
- [The Mythical Man Month](http://www.amazon.com/gp/product/0201835959 "Check it out on Amazon")
- [The Design of Everyday Things](http://www.amazon.com/gp/product/0465067107 "Check it out on Amazon")
- [The Pragmatic Programmer](http://www.amazon.com/gp/product/020161622X "Check it out on Amazon")
- [Surely You're Joking, Mr. Feynman](http://www.amazon.com/gp/product/0393316041 "Check it out on Amazon")
- [Defensive Design in the Web](http://www.amazon.com/gp/product/073571410X "Check it out on Amazon")
- [Refactoring: Improving the Design of Existing Code](http://www.amazon.com/dp/0201485672 "Check it out on Amazon")
- [Getting Real](http://www.amazon.com/gp/product/0578012812 "Check it out on Amazon")
- [Peopleware](http://www.amazon.com/gp/product/0932633439 "Check it out on Amazon")
- [The Lean Startup](http://www.amazon.com/gp/product/0307887898 "Check it out on Amazon")
- [Becoming a Technical Leader](http://www.amazon.com/gp/product/0932633021 "Check it out on Amazon")
- [The Inmates Are Running the Asylum](http://www.amazon.com/gp/product/0672326140 "Check it out on Amazon")
- [JavaScript: The Good Parts](http://www.amazon.com/gp/product/0596517742 "Check it out on Amazon") _(for the second, and **third** times)_
- [Don't Make me Think](http://www.amazon.com/gp/product/0321344758 "Check it out on Amazon")
- [Mastering Regular Expressions](http://www.amazon.com/gp/product/0596002890 "Check it out on Amazon") _(half of it, as it progresses in an excruciatingly slow fashion)_
- [Code Complete 2](http://www.amazon.com/dp/0735619670 "Check it out on Amazon") _(about a third, reading it on the bus made my hands hurt, and I gave up)_

It's been really refreshing to read books in print form, as during 2012 I read _mostly ebooks_, but that wasn't such a great experience. I could've gotten by without some of these. For example, _Getting Real_ was mostly a summary of things I already knew, but it was so easy to read that it didn't really hurt that much, and _Becoming a Technical Leader_ felt like a self-help book, so I couldn't recommend it, either. On the flip side, _The Pragmatic Programmer_ and _Surely You're Joking, Mr Feynman_ were **absolutely awesome** reads. _Programming Pearls_ was also a fantastic classic I really enjoyed.

> I already have a ton of books lined up to read in 2014. In particular, I'd like to read more about compilers, operating systems, and mobile development.

## Joining a Community

Ever since I started blogging I became much more aware of the community around me, I started using Twitter _(yeah, in 2013, I know)_, and following a lot more blogs, and a lot more actively than I had been doing. While I might argue sometimes that this might've hurt my productivity, I actually just came up with a solution to keep in check my addiction to staying up to date. [`hose`][16] is a command line tool that easily knocks off domains so you get to work.

If you're interested in joining the community at large, there's an [old blog post][17] which might help you get started. If you don't really know _what_ to read, check out [the RSS feeds I subscribe to][18]. I tend to update that file somewhat regularly.

I came into some issues some time after joining the community. Namely, I started obsessing about different social metrics I can't really act upon. Closely tracking this data is a waste of time and you should stop doing it. Here's an awesome blog post on the subject: [Managing a Mind][19].

> Next year, I'm hoping not to obsess over page views, stars, and followers.

## Becoming Heard

According to Google Analytics, this blog received roughly 100k page views in the March-December period, which feels pretty good for a first year, but I don't really have any data to compare its performance to. At some point in June I published [Uncovering the Native DOM API][20], which was, I think, the first article in this blog to appear on [JavaScript weekly][21], resulting in an exciting 1000 views in a single day. The follow-up article, [Getting Over jQuery][22], also did pretty good, becoming the most widely read article so far, with close to 10k visits, overall. Here's a graph showing page views over time.

![visits.png][23]

Popular blog articles are below, ordered by page views.

1. [Getting Over jQuery](/2013/07/09/getting-over-jquery "Check it out!") _~10k_
2. [Teach Yourself Node.js in 10 Steps](/2013/07/12/teach-yourself-nodejs-in-10-steps "Check it out!") _~8k_
3. [We don't want your Coffee](/2013/09/28/we-dont-want-your-coffee "Check it out!") _~7.6k_
4. [Uncovering the Native DOM API](/2013/06/10/uncovering-the-native-dom-api "Check it out!") _~5.8k_
5. [Grunt Tips and Tricks](/2013/11/13/grunt-tips-and-tricks "Check it out!") _~5k_
6. [Deploying Node apps to AWS using Grunt](/2013/09/19/deploying-node-apps-to-aws-using-grunt "Check it out!") _~3.6k_
7. [The Angular Way](/2013/08/27/the-angular-way "Check it out!") _~3.5k_
8. [9 Quick Tips About npm](/2013/12/14/9-quick-tips-about-npm "Check it out!") _~3.1k_
9. [Fun with Native Arrays](/2013/11/19/fun-with-native-arrays "Check it out!") _~2.8k_
10. [Continuous Development in Node.js](/2013/09/26/continuous-development-in-nodejs "Check it out!") _~2.4k_

This was definitely **an exciting first year** for the blog, and I hope to see the trend rise in 2014.

> In 2014 I'll strive to keep on writing and learning through educating myself as I investigate topics to to talk about on this blog.

## Contributing

Lastly, I'm glad about the contributions I was able to make to open-source throughout 2013. Most notably, [grunt-ec2][24], which lets you create EC2 instances and deploy Node applications to them in a _completely automated fashion_. In developing that Grunt plugin I learned a lot about Amazon Web Services and `nginx`, and that has been a tremendously rewarding experience for me.

In 2014, hopefully, my [book on JavaScript Application Design][26], which I started putting together in 2013, will go to print, and I'm also really excited about that, being my first book, and everything.

> Next year I'll try and participate in other people's open-source projects, in addition to developing my own.

Overall, 2013 was a great year where I learned a lot of things, and I hope 2014 is exactly the same in that regard. **Happy new year everyone!**

![ponyfoo.png][25]

Such pony.

  [1]: http://12factor.net/ "The Twelve-Factor App"
  [2]: https://github.com/bevacqua/node-assetify "Node asset management simplified"
  [3]: /2013/01/18/asset-management-in-node "Asset Management in Node"
  [4]: /2013/03/22/managing-code-quality-in-nodejs "Managing Code Quality in NodeJS"
  [5]: https://github.com/bevacqua/grunt-assetify "Compile your assetify static assets before even launching your application, it boasted"
  [6]: /2013/03/12/introduction-to-seo-and-content-indexing "Introduction to SEO and Content Indexing"
  [7]: http://substack.net/shared_rendering_in_node_and_the_browser "Shared rendering in node and the browser, by substack"
  [8]: /2013/03/06/defensive-design "Defensive Design"
  [9]: /2013/04/01/a-note-on-everyday-usability "A Note on Everyday Usability"
  [10]: https://i.imgur.com/5aOGBKJ.png "HTTP 400? That's an error!"
  [11]: https://github.com/bevacqua/ponyfoo/blob/master/src/hosts/blog/static/js/hooks/thin.validation.js "Response validation in NBrut"
  [12]: https://github.com/bevacqua/ponyfoo/tree/master/src/hosts/blog/static/js/nbrut "The NBrut client-side MVC framework"
  [13]: https://github.com/janl/mustache.js "Mustache.js on GitHub"
  [14]: https://github.com/bevacqua/ponyfoo/blob/master/src/hosts/blog/static/js/nbrut/nbrut.thin.js "The thin HTTP layer in NBrut"
  [15]: https://github.com/bevacqua/ponyfoo/blob/master/src/hosts/blog/static/js/views/user/profile.js "Sample view in NBrut"
  [16]: https://github.com/bevacqua/hose "Trap access to domains you use frequently, from 9 to 5"
  [17]: /2013/07/02/tech-news-reading-hints "Tech News Reading Hints"
  [18]: https://github.com/ponyfoo/linkdump/blob/master/rss-opml.xml "Feeds I follow, in opml-xml format"
  [19]: http://24ways.org/2013/managing-a-mind/ "Managing a Mind on 24ways.org"
  [20]: /2013/06/10/uncovering-the-native-dom-api "Uncovering the Native DOM API"
  [21]: http://javascriptweekly.com/ "JavaScript Weekly Newsletter"
  [22]: /2013/07/09/getting-over-jquery "Getting Over jQuery"
  [23]: https://i.imgur.com/0XfwM70.png "Visitors to Pony Foo over time, during 2013"
  [24]: https://github.com/bevacqua/grunt-ec2 "Create, deploy to, and shutdown Amazon EC2 instances"
  [25]: https://i.imgur.com/ElJqTpR.png "Happy New Year!"
  [26]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
