# Subscriptions via email

Early in 2013 I added a link on the sidebar so that readers could subscribe via email, in addition to using RSS readers. The grayed subscribers in the screenshot below were registered before the first redesign -- which revolved mostly around performance. In the beginning, you could subscribe only through the sidebar. A while after that I made it so that publishing comments for the first time, which asks for an email address to render a gravatar, gives you the option of subscribing to the emailing list as well. Later this year I added a second way of subscribing, right below articles, for people that don't bother checking out the sidebar. Lastly, I've put together [a landing page][2] where I can direct users who want to subscribe.

[![Subscribers to Pony Foo over time][1]][2]

Around a week ago, I made the graph in the screenshot above publicly accessible on the [Subscribe page][2], for the curious who asked. I had written it for my back-end a while back, but it looks pretty enough to be available to the public. I also thought it'd be a nice touch from a transparency standpoint.

As you can see, the most evident spike in subscribers in recent months coincides with the time when I started my daily blogging experimentation, and ramped up through the [ES6 in depth series][3] publications.

# Readership

Last year the blog had received roughly 250k page views. This year, it **served over 650k page views**, according to Google Analytics. Maybe next year it'll break through the _1 million page views_ barrier. What makes me hopeful in that regard is the significant improvement in the second half of the year, as revealed below.

![Page views last year on Pony Foo][4]

When it comes to individual articles, the [ES6 overview][5] went on to become the most widely read article on the blog yet, with 46k views. Here are the top articles by page views in the last year.

![Most viewed articles last year][6]

# Design and Maintenance

This year I've also spent some effort in redesigning the site to look and feel more like something designed this century.  
Before the redesign the site was quite the eye sore -- typical for your average blog, but I didn't write the entire codebase from scratch to settle for average!

![A screenshot of Pony Foo before the redesign][7]

I based off of the logo, kept the pink that has been stuck with the blog ever since it was created, and chose a few bright colors that are used throughout the site, giving it a somewhat more vivid appearance. You can read more about the redesign in [the article I wrote back then][9].

I also made the logo sharper, added an animation for it, and very recently made it pop out on the browser `console` and in an HTML comment on the site, too.

![A screenshot of the Pony Foo logo in its different formats][8]

Behind the scenes, I added the ability to schedule articles to be published and tweeted about at some point in the future, using a `cron` job. This turned out to be quite useful in allowing me to write articles in advance and then not have to worry about keeping the publication schedule in motion.

# Speaking, Open-Source, and Writing

In a more personal note, in 2015 I spoke [at nine conferences][11], compared to [six in 2014][10] _-- which was my first year as a public speaker_. Travel can be a lot of fun, meeting new people and going to faraway new places, but it also takes a lot of your time. A balance is hard to reach, but the experience makes it worth it.

Near the start of year I open-sourced [`dragula`][12] and it seemed to hit a nerve in people who just couldn't be bothered with drag & drop. [As I mentioned before][13], a combination of ease of use, documentation, and the lack of a pure drag & drop component that wasn't tied to a particular framework _(e.g Angular, React)_ nor a library _(e.g jQuery UI)_ drove `dragula` to formidable popularity. In just 8 months it received over 200 issues, support and feature requests, as well as over 70 pull requests.

[![The logo for dragula -- it's awesome!][14]][12]

When it comes to writing, I've been riding a emotional roller coaster ever since I hopped onto blogging. Inspiration comes and goes, as any writer would tell you. It goes more often than I'd like to admit, but when I feel inspired I get the idea to write insanely long article series, so I guess there's a good balance in there.

> At the moment I'm writing a second book, although that deserves an article of its own. I'm very excited for things to come next year. Particularly, _**I'm getting married soon -- so that's awesome!**_ I'm also going to be undertaking a new job, which will lead me to once again push my boundaries and explore unknown territories.
>
> Things that when appropriately leveraged, in my experience, are huge drivers of learning and innovation.

I wish I could say more about the book and my new job, but for now I'll just leave you with the look back on how far Pony Foo has come in three years.

Fine, the book is _obviously about JavaScript_, but unless you were in the audience at dotJS in Paris this month, I'll taunt you for a little while longer before I announce the book title and the topics that are covered. I'm really excited about this one!

[1]: https://i.imgur.com/zUeYXVl.png
[2]: /subscribe "Get Subscribed to Pony Foo!"
[3]: /articles/tagged/es6-in-depth "ES6 in Depth on Pony Foo"
[4]: https://i.imgur.com/PSB5DHD.png
[5]: /articles/es6 "ES6 Overview in 350 Bullet Points on Pony Foo"
[6]: https://i.imgur.com/ZHjbqw5.png
[7]: https://i.imgur.com/eihfWoU.jpg
[8]: https://i.imgur.com/ISt6ziQ.png
[9]: /articles/redesign "Pony Foo Gets a Face Lift on Pony Foo"
[10]: http://lanyrd.com/profile/bevacqua/2014/
[11]: http://lanyrd.com/profile/bevacqua/2015/
[12]: https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub"
[13]: /articles/why-i-write-plain-javascript-modules "Why I Write Plain JavaScript Modules on Pony Foo"
[14]: https://github.com/bevacqua/dragula/raw/master/resources/logo.png
