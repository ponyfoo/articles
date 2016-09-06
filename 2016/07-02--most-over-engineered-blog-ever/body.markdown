# Iterate. Always Iterate. And Then Some.

*There's one maxim that has driven Pony Foo since its inception: **iteration**.* I didn't want the project to go stale, so I forced myself to deploy what I had as soon as possible. As soon as I had a functioning blog. What good is a functioning blog without articles? Great question. I started writing articles in Markdown files before I was done with the engine, so that the blog wouldn't feel so empty upon launch. There was no big launch, I didn't care. It wasn't a product, it was just for myself. Sure it had some words on it, but it's not like anyone was going to read them yet.

When I first began working on the code, Pony Foo consisted of a client-side Markdown editor that `POST`ed to an `express` app. There was no authentication. It created articles, and you could visit those articles. Right before launch, I added authentication to prevent unsuspecting users from stumbling upon the oozing pile of mud that I called an admin panel. There were no comments. Just article CRUD _(create read update delete)_ views. The home page consisted of a `for..of` loop rendering every article in its full glory. That was it. That was a lot.

That simple bundle of code would be my first venture into a product of my own devising. It leveraged [`pagedown`][pd] and little else on the client-side. On the server-side, however, I had learned about Node.js and MongoDB. I was using `express`, `jade`, and `mongoose`. I had learned about CJS. Back at work I used C# and Microsoft technologies, I had never known much else.

Here's an old pic of how the site looked like back then. There was some sort of REST API. In this screenshot, the engine is pretty "mature", in that it has input validation, comments, authentication, and a bit more of stuff.

![Primitive ancestor of Pony Foo][oldpic]

Did I mention I also implemented my own MVC framework for Pony Foo? Yes, it was client-side rendered only. No, it wasn't very good. Most importantly, it taught me a lot about the internals of these kinds of frameworks. It taught me enough that I could follow the code around when later in my career I'd have to plunge into AngularJS's internals. It was a valuable learning experience, regardless of its material output back then.

Due to my client-side rendering framework I had to learn about PhantomJS so that my blog could be crawled by robots. I even wrote [an article][seoamuse] about it. I wrote about everything I was doing on the blog back then. Which was great. For many reasons. It helped me think about the things I was doing. Eventually, as my articles got better and more interesting, it helped other people out as well. This wasn't something that came together in a day or two, nor would I have had much to share if I just fired up Medium and started ranting about `<marquee>` tags. I had something to share because I had so much to learn. Here's a picture of how SEO worked back then.

![SEO in the old days of Pony Foo][seopic]

It was a terrible idea. It was an experiment. I learned from it. I wrote some more. I iterated.

I learned lots about UX, too. I learned people didn't care at all about my site. Definitely not enough to create an user just to post a comment telling me that they didn't care about my site. So I removed authentication. That brought on spam robots. I had to learn how to deal with those, too. I implemented OpenSearch, so that people didn't need to find the search box on my blog. Guess [how search worked][search]? I applied a `RegExp` on the query and then handed that off to MongoDB. Brilliant!

I had a fully-functioning blog with articles and comments, running on Heroku. My Heroku instance loved running out of memory, and at some point I pretty much rebuilt the whole thing from scratch. This was a complete rewrite, but it paid off. It was just a side project. I kept writing articles, so it wasn't such a big deal. The rewrite got out of hand, I envisioned a developer-oriented platform where everyone could publish their own articles. Something like Ghost and Medium before they existed, but much more terrible. Eventually I got back on track and skimmed the unnecessarily complex features.

Still, I was in the middle of a rewrite, and the client-side rendering thing was so bad, I completely rewrote that. Taunus -- a shared-rendering MVC engine -- was born. It's not my greatest creation, but it has noble intentions at heart. I was able to reuse Jade views and server-side controllers, which blew my mind. As the comments indicate in [the blog post announcing Taunus][taunuspost], the blog became way faster now that it was rendering views server-side. I had been experimenting with `grunt`, `gulp` and `npm run` for a long time. My musings on the build tools became quite popular articles, one of which spawned the first talk I ever gave *-- at [JSConf US 2014][jsconf]*.

While enumerating technologies may be a trite endeavor, explaining how some aspects of the blog were developed may be more informational. **Let's fast-forward to where Pony Foo is today** _-- feature-wise --_ to get a better glimpse into why I say its the most over-engineered blogging website ever.

# Text, Markdown, and HTML

Text manipulation deserves its own corner. There's so many things Pony Foo does to text.

- Emoji are expanded into images using Twitter's [`twemoji`](https://github.com/twitter/twemoji "twitter/twemoji on GitHub") library 💥
- Text is modified so that two or three dashes turn into a longer one ---, quotes are "prettified", and so on...
- A custom Markdown plugin allows me to add `<mark​>` tags so that I can highlight arbitrary code for better readability, as a teaching tool: `[<mark>...</mark>'important']`.
- There's code to extract images from HTML, to be used when filling out `<meta>` information
- In the same vein, `<meta>` descriptions are usually pulled from specific places, but also summarized via [`trunc-html`][thm]
- A few things like tweets or CodePen links can be embedded into articles for a more interactive and less boring blog-reading experience
- Relative links are made absolute when they need to be sent over email, so that the same code can be reused when accessed somewhere other than at ponyfoo.com

# The Lifetime of an Article 🗞

Everything begins with a draft. Earlier on, articles consisted of a title and a body, but not much else. Over time more metadata was incorporated, such as tags, auto-computed related articles, and the ability to get an auto-generated slug based on the title or pick a hand-crafted one, _such as: [`/articles/most-over-engineered-blog-ever`][moe]_.

Tags are entered using an input based on Stack Overflow's UX for tag editing, with an autocomplete feature as well. These two features are split into two open-source packages: [`insignia`][nsg] for tag editing and [`horsey`][sey] for autocomplete. The screenshot below shows the tag editor at work. Tag suggestions are merely a compilation of previously-used tags, merged with "well-known tags". That is, tags which have been explicitly inserted into the database alongside a description and a title.

![Tag editing for the Article Composer][tagedit]

When displaying search results, the *"well-known tags"* are also looked up and any such matches are explained to the user. This can be useful for context. For example, the ES6 in Depth article series displayed below, wherein I briefly describe what the tag entails. The description is terse, but the functionality is there and -- over time -- I can update the descriptions to make them more accurate.

[![Search results page for the "[es6-in-depth] [es6]" query.][tagsearch]][indepth]

For the most part, I write pieces of Markdown like the summary, which goes below the title; the first part of the article, which is the "cut off" point when sending out an email teasing readers to visit a freshly published article; and the main body, containing the rest of the article. Then there's optional blocks of text, like _"Editor's notes"_ when someone writes up a guest post and I'd like to leave a note. As the contents of the article are written, the front-page summary used in headlines is compiled and displayed in a live preview. If the summary feels disjointed I can write up a different version of the summary, that's not included elsewhere in the article but helps explain what the article is about.

> Drafts can be saved for later, updated, and shared. Drafts are shared by appending a `?verify=$HASH` token to the article, and anyone who knows the `$HASH` can read the draft.

Drafts can be scheduled for publication at a date in the future, -- via a `cron` job -- or published immediately. Lots of things occur when you publish an article. For example, Elasticsearch is used to find similar articles so that we can display them as "related articles". There are options to share the article over several mediums upon publication.

![Sharing an article upon publication][share]

Each medium has *its own* particularities.

**The email** goes out to all subscribers. Emails use Jade and Stylus to render HTML and CSS, like everything else in the blog. I've always used API-based services for email, but I still developed [`campaign`][campaign] as a way to make it easier to send those emails in an agnostic way. The subscription model was built into Pony Foo itself. In the short term, this was nuts. In the long term, this allowed me to reuse the same code when I came up with [Pony Foo Weekly][pfw], which was the good kind of nuts. Emails adhere to Google's [`JSON+LD` markup][ld] to describe actions, meaning that when we send you an email you see the quick action buttons in the right hand side of the following screenshot.

![Emails and their quick actions][emails]

Emails have branding that matches the website, and there's an unsubscribe button on every email _-- which is also pretty important._

![An email about a Pony Foo Weekly submission][emailexample]

**The tweet** includes a [Lead Generation Card][leadgen] that allows Twitter users to subscribe via email with a single mouse click. There isn't much of an API to pull the leads generated by Twitter, so I had to write a module _-- [`twitter-leads`][lead] --_ which visits the Twitter website, authenticates, downloads a `.csv` file, and compares its entries with my records to see whether any new users have clicked on the Twitter card. In my own playful fashion, tweets are brimming with emoji, and when possible _(as determined by Twitter's character limit)_ include a couple of tags, the author's Twitter handle and a few other words besides the article's title and a link to it. All of the above is just a means to an end: *not be another super dull automatically generated tweet.*

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">📰 Just published!<br>🍧 The Double-Edged Sword of the Web<br>✍ By <a href="https://twitter.com/nzgb">@nzgb</a><br>🏷 <a href="https://twitter.com/hashtag/opinion?src=hash">#opinion</a> <a href="https://twitter.com/hashtag/web?src=hash">#web</a><br>➡️️ <a href="https://t.co/HnbWfxcPrR">https://t.co/HnbWfxcPrR</a><a href="https://t.co/gaRPiM8UFq">https://t.co/gaRPiM8UFq</a></p>&mdash; Pony Foo (@ponyfoo) <a href="https://twitter.com/ponyfoo/status/750302739981271040">July 5, 2016</a></blockquote>

_When you [look at that tweet on Twitter.com][tweet], the link for the Lead Generation card expands into the following:_

> ![Pony Foo's lead generation card][leadcard]

Before the Tweet loads its `<iframe>`, tweets are styled in such a way that they somewhat resemble the original Twitter experience, for added progressive enhancement.

![Tweet before loading the script.][tweetpro]

Sharing **on Facebook** is largely a boring affair, if not for the [overly complicated process][fbtok] one has to endure in order to obtain a long-lived access token for a Facebook Page _-- with posting rights._

When it came to [EchoJS][ej], [Hacker News][hn], and [Lobste.rs][lob], it was mostly a matter of using `request` and a cookie jar to persist a session where I'd visit each site, authenticate, and submit a news item. While no doubt some of you will consider this to be a spam tactic, I generally only share content relevant to each aggregator, and doing it manually involved tons of repetition or wasted time.

Sometimes, things can go wrong. Credentials are invalidated, API calls fail, services let you down. Sometimes, you just need to test something out. The article review interface lets me reshare published articles via any medium. If something goes wrong during publishing, I can typically just click on the relevant share button and get a free retry. In case you're wondering, the white email button sends an email only to myself. That's typically useful when I'm testing something out on the layout. The black email button, meanwhile, sends the email out to every subscriber.

![Article sharing on the Article Review page][artshare]

Automating all of these small tasks away allows me to schedule an article, to be published at some point when I'm away from the keyboard, and still share it through relevant news sites.

# Conference Talks, Open-Source, Books, and so on

The [Speaking][speak] page was a fun one to implement. First off I had to add a database model for presentations and speaking engagements. That was boring, but when that was out of the way, I got to play around with the Google Maps API. I wanted the experience of that page not to suck, so I compute the URL to [a static map][statmap], like the one below, while I load an interactive map in the background.

![An static representation of places where I gave conference talks][staticmap]

I used custom location markers in shades of pink, signaling that an event is in the past or the future. When you click the prompt to interact, a fully interactive map like the following fades into view. This map uses a different _(slower)_ API, but the map can be interacted with.

![An interactive map of places where I gave conference talks][interact]

On that same page I list [every presentation I gave][present], along with their slides, the talk description, the video filmed at a conference, and some helpful resources related to the presentation.

The [open-source section][oss] doesn't do much of anything other than show a list of projects I enjoyed working on, although it does pull star counts from GitHub _-- but I have a cooler standalone project that leverages the GitHub API: [`hubby`][hubby]._

The [Books][bk] section isn't that interesting either, _-- and there's only a single book there at the time of this writing --_ but it's still neat to have everything in the same website. I used to have [bevacqua.io][bvq] for "personal" stuff, but ended up merging both websites into Pony Foo.

> Overall, these sections let me play with one API or another, but there wasn't much else in the way of technical challenges to implement them.

When I wanted to invite people to subscribe to Pony Foo, I put together a small landing page at [`/subscribe`][subscribe]. That page included a `d3` graph of subscribers and page views, which helped me learn about `<svg>` and the basics of `d3`. That was a super fun way to learn `d3`, and useful at the same time!

[![A graph of subscriber and page views][subscribegraph]][subscribe]

# Pony Foo Weekly

I'm a huge fan of weekly newsletters. I'm subscribed to over a dozen of them and I skim most of them every week for great content. The problem I saw was that most of those newsletters cover a specific area, while none of the covered interesting things around the web platform. Wanting to seize that opportunity, I decided to launch [Pony Foo Weekly][pfw].

I implemented the features you commonly see in other newsletters: a home page where you can easily subscribe, a detailed history containing every newsletter issue ever published, an RSS feed, and a sponsorship page. Links in each issue may have an image, some Markdown, and a few tags. One detail I try to keep adding to issues is the Twitter handle for the author of each link, because you may find the people behind the content even more interesting than their content. There's also the possibility to insert custom headlines, many colors, emoji, and other colorful artifacts. 🎨

> I spent *\*a lot\** of time making sure that newsletter issues are displayed properly on email clients, RSS readers, mobile devices, and on desktop. Issues are rendered using the same HTML everywhere, and sometimes slightly different CSS. This helps avoid missing information and keeps me DRY.

When it came to sending the actual emails, I slightly rewrote the subscription code I already had. I needed to add support for different subscription types, so that readers who wanted to get either only articles or only the newsletter could do that if they didn't want both things.

As for scheduling the newsletter, it's pretty straightforward. You add different sections to a newsletter, as we'll see later, and save your drafts. When an issue is saved, it can be marked as "ready", which means it'd be okay to send it out to our subscribers. When an issue is marked ready, it may be picked up by a `cron` job and sent out to subscribers.

Every wednesday, patrons get the newsletter issue; that's a day earlier than the general population, who get issues on thursdays. This wasn't complicated to implement, and its nice to show appreciation to those who appreciate your work! 💞

Then there was the sponsorship material. I started out lean, with a media kit describing the different sponsorship opportunities. I stayed conventional and just offer a few different ways to present your links, with different price tags. Still, I spent a good chunk of time making sure the media kit looked good and professional, as it'd be the main means of income for Pony Foo.

Here's one page of [the media kit][mk]:

![Part of our Pony Foo Weekly media kit][mediakit]

Recently, I realized how inconvenient it was to submit content for Pony Foo Weekly. You would need to learn that there were [tips@ponyfoo.com][tips] and [sponsor@ponyfoo.com][sponsor], and send emails our way to ignite a conversation. This was a very manual process that not everyone is willing to go through, so I decided we could do better.

We just launched the [Submissions][sub] page. This page is comprised of a massive form divided into three sections. Each section is fairly simple. In the first one, you can specify the kind of submission you're trying to make _-- telling us whether its a suggestion or a sponsored link request --_ and you need to paste your link. The submission type is optional and can be omitted, but we'll need the link to proceed! Still, we ask for that because its also a chance to explain our sponsorship offerings _sort of unobtrusively._

![The first section of the Submissions page.][sub1]

Then we scrape that link, show you any relevant information we scraped off of the site, and let you edit that. There's a live preview where you can verify that the submission looks great so that we both love it.

![The second section of the Submissions page.][sub2]

Finally, we want to gather some information about you, such as your name, your email, and any comments you may have. If you picked a sponsored submission, we'll also ask how many times you'd like us to feature your ad, a few dates you'd like to suggest, and whether you'd like an invoice. Confecting an invoice can take a lot of time, so automating them away can be a huge win. 

![The last section of the Submissions page.][sub3]

Once you send your submission, you'll get an email with a special link that you can use to edit the submission at any time. We'll be copied on that email, so a conversation can start immediately. I can let people know whether we'll take their suggestion or not, or we can quickly confirm the sponsorship request and get their content on the next newsletter.

When a submission is accepted on the back-end, another email is sent to everyone involved, letting the submitter know their submission was accepted. If the submitter is a sponsor, they might get an invoice, if one was requested. There was already an invoicing system on Pony Foo, so this feature took that one step further by creating an invoice, visiting that page via `phantomjs`, printing it to `.pdf`, and attaching it to the acceptance email.

*The attached invoice looks like the example below. It is generated and attached automatically. 🎉*

![An invoice example.][invo]

**But wait, there's more.**

We also adapted the link submissions form into [a browser extension][pwe] _-- currently available for [Chrome][cws], [Firefox][amo], and (soon) [Opera][aoc] too._ The extension is a bit simpler, so that it gets out of the way while people are browsing the web.

It will ask for your details *(name, email)* only once, as a lean *"initial setup"*. It also tries to be helpful by scraping the websites and displaying a real-time preview, just like the Submissions page. For now, it's limited to suggestions and you can't specify that you'd like to sponsor Pony Foo Weekly, although there's a comment field. In the future, we may add the ability to submit sponsorship requests directly from the browser extension. *For now, it works quite well!*

You can use it to quickly submit links as you come across them on the web, without having to hop onto the [Submissions][sub] page.

[![Pony Foo Weekly Link Submission Extension][extension]][pwe]

Experimenting with browser extensions is always awesome. There's just so much you can do! Okay, but **seriously** _-- there's more._

# Roles and Contributing Authors

A bit before introducing Pony Foo Weekly, I added support for user roles. Before that, it was just me, but I planned to let others help out with [Pony Foo Weekly][pfw]. Adding roles was surprisingly easy, and obviously useful. When I had roles, adding authors to articles was also fairly simple, there was already an `author` field on articles, so expanding that and displaying their name on the article's trivia box wasn't particularly hard.

I expanded on that notion a bit by adding a page, explaining how you can [become a contributor][join]. With that came user profiles, and little things like Twitter handles and a personal website.

Incrementally, but surely, Pony Foo becomes a platform. If you'd like to write articles for us, or to contribute to Pony Foo Weekly on a regular basis, just shoot us an email.

_You can learn more in the [Join Our Team page][join]._

# A Cohesive Back-End

What else could possibly be in the back-end that I haven't talked about yet? Oh, boy. There's the logs, obviously. This is just a list of events _-- and some paging._ It makes it easy to skim through the logs without going through the raw database logs when we need to check if something is amiss.

![Some sample logs.][logs]

There are a few other views that give insight into subscribers, or to send announcements via email, although I've probably only used the announcements feature once or twice!

> With the update of user roles, I finally implemented a users CRUD page, which I had resisted for so very long _(over three years!)_.

As I brought up earlier, you can create invoices on the site. While this is a fairly primitive page, all I cared about was the template, and it beats having to edit HTML by hand, so I finally gave in and implemented the database models and views needed for an invoicing feature. When I decided to also implement automated invoicing for sponsors through the [Submissions page][sub], the invoicing feature *turned out to be really helpful*.

There are CRUD pages for open-source projects, speaking engagements, talk presentations, user profiles, but none of these are interesting. The articles composer was mostly covered earlier. The last piece of Pony Foo I'd like to discuss is the "Newsletter Assembler", this is the thing I wrote to make building email newsletter issues less cumbersome.

It's pretty "fun" to use _-- as far as chores go._ This was one of my main goals, as I didn't want Pony Foo Weekly to feel like an annoyance, because I'd have to compile these newsletter issues every week. It looks like this:

![Pony Foo Weekly Assembler][assembler]

Each weekly issue is largely just an array of "sections". The `<textarea>` that you see in the beginning is meant to be a greeting, and then there's the "sections". A section can be a heading, a link with all of its supporting information, a piece of CSS _(compiled through Stylus)_, or an arbitrary piece of Markdown. The yellow toolbox lets you click or drag any section into somewhere on the main area, where we compose the newsletter issues. User-provided submissions that were accepted are displayed in the sidebar as well, on the purple toolbox. When those submissions are clicked or dragged into the newsletter issue, they become links containing all the information that was contained in the submission.

In the screenshot above, the heading sections are expanded, letting you edit the text, the background color, or the foreground color. You can drag and drop any section anywhere else on the editor, making it easy to move sections out of the way and to sort links. This is particularly useful for the weekly newsletter use case, where you're constantly trying to figure out which links are more important and which ones deserve to be closer to the top.

Each section can be expanded, just like the link section shown below. The information here is largely the same as what we ask about in the [Submissions][sub] page. You can pick a tag by clicking on them or by typing it out. There's a little thumbnail preview of the image, for convenience, and you can duplicate sections to give yourself a head start to type things out faster.

![An expanded link section][expa]

After the editor there's a live preview of how the newsletter currently looks like, and a save button. Yay! 💌

If you're still reading this and you haven't subscribed yet, you [probably should subscribe][subscribe]. Did I mention there's a chart of page views and subscribers on that page?

# Component Libraries

There's was a lot of code involved, too. Here are some examples. Some of you may call the following the _"Not Invented Here" syndrome_. I call it experimenting and figuring out how things work. That's why most of us are in engineering, right?

- [`woofmark`](https://github.com/bevacqua/woofmark "bevacqua/woofmark on GitHub") is the Markdown editor you see everywhere on the site
- [`megamark`](https://github.com/bevacqua/megamark "bevacqua/megamark on GitHub") is the Markdown compiler, using [`markdown-it`][markit] under the covers but also adding the code highlighting and `mark` convention
- [`insane`](https://github.com/bevacqua/insane "bevacqua/insane on GitHub") is an HTML sanitizer preventing unwanted tags or attributes from making it through to raw HTML
- [`domador`](https://github.com/bevacqua/domador "bevacqua/domador on GitHub") is used in some places to turn HTML back into Markdown
- [`dominus`](https://github.com/bevacqua/dominus "bevacqua/dominus on GitHub") is a copy of `jquery`, where the `Array` prototype in an `<iframe>` is extended _(via [`poser`](https://github.com/bevacqua/poser "bevacqua/poser on GitHub"))_ to get a rich API
- [`campaign`](https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub") is used to send out emails, with their corresponding layouts and all else
- [`baal`](https://github.com/bevacqua/baal "bevacqua/baal on GitHub") is a clone of the deployment process used by [`ponyfoo`](https://github.com/ponyfoo/ponyfoo "ponyfoo/ponyfoo on GitHub"), it's automated, auto-scaled, load-balanced, immutable, and awesome
- [`dragula`](https://github.com/bevacqua/dragula "bevacqua/dragula on GitHub") is used for the drag and drop feature in the newsletter editor
- [`rome`](https://github.com/bevacqua/rome "bevacqua/rome on GitHub") is used to pick a date and a time in a few places
- [`insignia`](https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub") is used to prettify tags in the article editor
- [`horsey`](https://github.com/bevacqua/horsey "bevacqua/horsey on GitHub") is used to autocomplete tags in the article editor
- [`hget`][hget] is used to support plain-text requests to ponyfoo.com
- [`jadum`](https://github.com/bevacqua/jadum "bevacqua/jadum on GitHub") is a bit like `jade` when compiling views into a JavaScript function, except it understands context and uses `require` statements
- [`contra`](https://github.com/bevacqua/contra "bevacqua/contra on GitHub"), [`sluggish`](https://github.com/bevacqua/sluggish "bevacqua/sluggish on GitHub"), [`omnibox`](https://github.com/bevacqua/omnibox "bevacqua/omnibox on GitHub"), [`queso`](https://github.com/bevacqua/queso "bevacqua/queso on GitHub"), and [`ruta3`](https://github.com/bevacqua/ruta3 "bevacqua/ruta3 on GitHub") are some examples of reimplementing large libraries aimed at `node` for use in the browser
- [`taunus`](https://github.com/taunus/taunus "taunus/taunus on GitHub") is the MVC framework I wrote and still use _(but wouldn't recommend)_ for Pony Foo

<sub>_You can find all of the above on [my GitHub profile][beva]._</sub>

I already covered most of the [build][cp] [process][il] [in][ta] [older][fp] [articles][at].

Did I mention I also wrote several scripts around the main Pony Foo app? One such script creates the `<svg>` logo in several different formats: *`.svg`, `.min.svg`, `.png`. While this was a bit annoying to implement, it helped me create a variety of logos around the original as a way of enriching the brand. At the same time, it was super fun. At the moment, the only logo that's actively used on the site, besides the original and its adaptations for different browsers, is the one that has a christmas hat. I should probably do the same for halloween and other popular holidays, but I haven't iterated the logo that far yet. One problem here is that the logo atop the site contains a flashing pink animation that makes it pretty awesome, but at the same time its hard to repurpose that animation for different patterns on top of the original black and green logo. Over time, I'm sure I'll iterate to include the animation in my scripts as well, somehow!

There's also the following spritesheet.*

![Pony Foo Logo Flavors][all]

In a similar vein to automated logo generation, I came up with a CLI utility -- [`shots`][shotsrepo] -- which goes through the Internet's Archive Wayback Machine and generates a gif of the historical snapshots of any website. It's quite fun to watch how a site evolves over time in its historical `.gif` form, but `shots` takes a very long time to download all the images and then compile a `.gif` animation.

I probably won't evolve `shots` much further than its current form, but its a nice addition to the variety of ways in which you can visit Pony Foo. Particularly when we factor in [`hget`][hget]. As a command-line tool, `hget` can render any website in plain text. As a programmatic API, `hget` helps Pony Foo render itself as plain-text if the user-agent requests that as opposed to HTML. It works by loading the site through an HTML parser, extracting relevant text and links to images, and printing that out.

![Pony Foo in one .gif][snap]

Another cool thing I wrote was [`promisees`][pseeact] [_(source)_][psee], a web playground where you can enter arbitrary pieces of JavaScript and get back a cool visualization about Promises. I wrote this tool to help readers visualize examples in [a tutorial about promises][pid]. I was able to learn a bit more about `d3` while building Promisees, just like back when I built the [Subscribe][subscribe] page for Pony Foo.

Promisees is able to take snapshots of the `<svg>` element and then build a `.gif` recording -- entirely in-browser -- using [`vectorcam`][vc], a library I devised just for Promisees. Vectorcam serializes the current state of an `<svg>` element into a data URI, and then compiles each snapshot into a `.gif` using [`gifshot`][gshot]. Implementing an in-browser `<svg>` to `.gif` library was a super fun experiment!

[![Promisees in action][psees]][pseeact]

I built *quite a few* other things in my spare time, but I won't bore you with any more of them. I'll be **genuinely surprised** if anyone got this far down in the article. I swear I tried my best to keep the article interesting!

# Conclusions

With everything I described today, the most important take away is that none of this happened over night. Pony Foo is over three years old, and I iterated my way into its current state piling up small features. I implemented many libraries that already existed, as others before me, because I wanted to learn about designing and documenting those kinds of pieces of code. I built my own interfaces and social campaign features because I wanted to learn a bit more about marketing as well.

During my iterations I've learned about many things that interested me: *Node.js, deployments, UX, design, etc.*

Having a side project like Pony Foo helps drive my interest for the web platform, as well as my self-interests, of course. Side projects may also, *over time*, garner a bit of visibility and help you build an online presence. Pony Foo has quite a bit of expenses, but ads and donations make up for that.

> Sharing my knowledge with others has been, *and will continue being*, an invaluably rewarding experience.

I'm glad this side project started out as a blog, because that meant it was easy to release a first version I could then iterate upon. I'd love to hear what are some cool side projects you've been working on, or thinking about. If you were just thinking about a side project, why not get started?

*It's a fantastical adventure!*

[pd]: https://github.com/ujifgc/pagedown
[oldpic]: https://i.imgur.com/0kiHzgx.png
[seoamuse]: /articles/introduction-to-seo-and-content-indexing "Introduction to SEO and Content Indexing on Pony Foo"
[seopic]: https://i.imgur.com/JdHh6JF.png
[search]: https://github.com/ponyfoo/ponyfoo/blob/a02e1a129e5c4f293d902f91931364dd2fd76b33/src/controllers/api/1.0/entry.js#L171 "Search code back in 2013"
[taunuspost]: /articles/taunus-micro-isomorphic-mvc-framework "Taunus: Micro Isomorphic MVC Framework"
[jsconf]: /presentations/front-end-ops-tooling "Front End Ops Tooling Slides and Talk Video on Pony Foo"
[moe]: /articles/most-over-engineered-blog-ever "Why I Built the Most Over-Engineered Blog Ever"
[share]: https://i.imgur.com/JXQ3qPH.png
[leadgen]: https://business.twitter.com/en/help/campaign-editing-and-optimization/optimizing-for-leads-campaigns.html "Optimizing for leads"
[campaign]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
[pfw]: /weekly "Pony Foo Weekly: A newsletter about the open web, highlighting the most important news about the web every thursday."
[emails]: https://i.imgur.com/Y4eaeuW.png
[ld]: https://developers.google.com/gmail/markup/getting-started "Getting Started with Email Markup for Gmail"
[lead]: https://github.com/bevacqua/twitter-leads "bevacqua/twitter-leads on GitHub"
[fbtok]: https://github.com/ponyfoo/ponyfoo/blob/bf7f0ad613bac549479a50377a6c20c50f5266c7/scripts/get-long-lived-fb-access-token.js#L18-L26 "Instructions on getting a long-lived Facebook Page access token"
[ej]: http://www.echojs.com/ "JavaScript News Aggregator"
[hn]: https://news.ycombinator.com/
[lob]: https://lobste.rs/
[tweet]: https://twitter.com/ponyfoo/status/750302739981271040 "Pony Foo on Twitter"
[tweetpro]: https://i.imgur.com/s4B5zHq.png
[emailexample]: https://i.imgur.com/eAmEC0i.png
[speak]: /speaking "Conference Talks presented by Nicolás Bevacqua"
[statmap]: https://developers.google.com/maps/documentation/static-maps/ "Google Static Maps API"
[staticmap]: https://i.imgur.com/5auEGa1.png
[interact]: https://i.imgur.com/yUJF3FI.png
[present]: /presentations "Conference Presentations on Pony Foo"
[hubby]: http://bevacqua.github.io/hubby/?bevacqua "Hubby: A breeze of awesomeness in your browser."
[oss]: https://ponyfoo.com/opensource "Open-Source on Pony Foo"
[bvq]: http://bevacqua.io
[bk]: /books "Books on Pony Foo"
[mediakit]: https://i.imgur.com/XEHluHA.png
[sub]: /weekly/submissions "Pony Foo Weekly Submissions"
[sub1]: https://i.imgur.com/gLmYuyj.png
[sub2]: https://i.imgur.com/3OSp1qO.png
[sub3]: https://i.imgur.com/M4NjclX.png
[mk]: https://ponyfoo.com/weekly/sponsor "Sponsor Pony Foo Weekly"
[tips]: mailto:tips@ponyfoo.com
[sponsor]: mailto:sponsor@ponyfoo.com
[invo]: https://i.imgur.com/FZCqqJr.png
[join]: /contributors/join-us "Join the Team at Pony Foo"
[logs]: https://i.imgur.com/ECJFKDb.png
[assembler]: https://i.imgur.com/Ygmg953.png
[expa]: https://i.imgur.com/F3VE7HY.png
[subscribe]: /subscribe "Subscribe to Pony Foo!"
[beva]: https://github.com/bevacqua "@bevacqua on GitHub"
[cp]: /articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo"
[il]: /articles/inlining-critical-css "Inlining Critical CSS for Dynamic Web Apps on Pony Foo"
[ta]: /articles/talk-about-web-performance "Let’s talk about Web Performance on Pony Foo"
[fp]: /articles/fixing-web-performance "Fixing Performance in the Web Stack on Pony Foo"
[at]: /articles/tagged/performance "Your query [performance] yielded some results!"
[all]: https://i.imgur.com/hInzEc2.png
[snap]: https://rawgit.com/ponyfoo/ponyfoo/master/resources/shots/output/1024x768.gif
[nsg]: https://github.com/bevacqua/insignia "bevacqua/insignia on GitHub"
[sey]: https://github.com/bevacqua/horsey "bevacqua/horsey on GitHub"
[tagedit]: https://i.imgur.com/3Ei9TRk.png
[tagsearch]: https://i.imgur.com/4cbDvmq.png
[indepth]: /articles/tagged/es6-in-depth "ES6 in Depth articles on Pony Foo"
[artshare]: https://i.imgur.com/aUxxPmO.png
[leadcard]: https://i.imgur.com/Qw48vQI.png
[psee]: https://github.com/bevacqua/promisees "bevacqua/promisees on GitHub"
[pseeact]: https://bevacqua.github.io/promisees/ "Promise visualization playground for the adventurous"
[psees]: https://i.imgur.com/2L3WGrA.png
[pid]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
[shotsrepo]: https://github.com/bevacqua/shots "bevacqua/shots on GitHub"
[thm]: https://github.com/bevacqua/trunc-html "bevacqua/trunc-html on GitHub"
[markit]: https://github.com/markdown-it/markdown-it "markdown-it/markdown-it on GitHub"
[subscribegraph]: https://i.imgur.com/8ZzFE6C.png
[hget]: https://github.com/bevacqua/hget "bevacqua/hget on GitHub"
[vc]: https://github.com/bevacqua/vectorcam "bevacqua/vectorcam on GitHub"
[gshot]: https://github.com/yahoo/gifshot "yahoo/gifshot on GitHub"
[extension]: https://i.imgur.com/AJVSmfu.png
[pwe]: https://github.com/ponyfoo/ponyfooweekly-extension "ponyfoo/ponyfooweekly-extension on GitHub"
[cws]: https://chrome.google.com/webstore/detail/pony-foo-weekly-link-subm/ldiaebhociegnbdjbkfbhbpebeiamldi "Pony Foo Weekly Link Submissions extension on Chrome Web Store"
[amo]: https://addons.mozilla.org/en-US/firefox/addon/ponyfooweekly/ "Pony Foo Weekly Link Submissions extension on addons.mozilla.org"
[aoc]: https://addons.opera.com/extensions/details/pony-foo-weekly-link-submissions "Pony Foo Weekly Link Submissions extension on addons.opera.com"
