# Articles

I moved details people rarely care about _(like comment count, reading length, or tags)_ onto the sidebar, and emphasized the article's title, its first paragraph, and code blocks.

## Code Blocks

When it comes to code blocks, I adopted a more colorful and modern color scheme: _Tomorrow_. I also made `<mark>` tag highlights less invasive, as they were too distracting before the redesign, and many commented on how obnoxious that was.

![A code block using the Tomorrow syntax highlighting theme][1]

## Headings

A while before the redesign I had added anchors to every heading, so that you could go to an URL like [`/articles/es6-modules-in-depth#conclusions`](/articles/es6-modules-in-depth#conclusions "ES6 Modules in Depth, #conclusions"), but you really had to have intimate knowledge about the blog to even know you could link to headings like that. Now you can just hover over an anchor and click on it, ta-da!

![A heading being hovered over][2]

## Comment Threads

Then there were comment threads, those were pretty confusing as there were almost no visual cues differentiating replies in a comment thread versus an entirely different comment thread.

![The old comments design][3]

The current design makes an effort to illustrate the hierarchy behind comment threads by adding an imperceptible visual cue to the left of each comment.

![The new comments design][4]

The comment editor also got a face lift. It's now using [`woofmark`][5] and I think it looks much cleaner now. Let me know your thoughts, though!

## Licensing

Lastly, there's the **licensing**. I've been asked multiple times about whether my content was up for distribution or whatnot, and even though I _almost always said yes_, I'm sure some people thought they couldn't use the content and didn't bother to ask. That's why today I'm adding an explicit note stating that the content distributed on ponyfoo.com is [CC-licensed][6] -- at the bottom of every page in the blog.

[![The license footer][7]][8]

When I was done with the single article experience, I moved on to the rest of the site.

# Buttons and Links

This one bothered me for a very long time. Links looked very fat. Buttons were completely inconsistent, no two buttons looked the same, for some fringe reason. I took care of that, links aren't fat, and buttons have a cool background powered by _the same CSS_ that animates the logo. How sweet is that!?

![Some links and a button][9]

I guess while I'm on the topic I can comment on how I added `confirm` steps that prevent me from deleting drafts I've been working on for days. Yes, that happened, and it wasn't any fun at the time. I would add an Undo feature if it was user-facing, but since only I see it, the old `confirm` dialog is good enough!

# Home Page

The home page is probably the second most important page of the site. Even though each article is the most important page of the site, the home page is obviously _-- far and away --_ the most visited page in the site. Throughout the history of Pony Foo, the home page was basically a collection of individual articles one after the other. This didn't really work out great, as people couldn't effectively find anything other than the first few articles. There was so much information about each article that you quickly lost focus as to what you were looking at.

![An older version of the home page][10]

Don't even get me started with the first iteration of the home page. Remember these little circles looping around? **Ugh!** Such shame.

![The first iteration of the home page][11]

If anything, the new home page looks sort of like a news paper. Except for the bazillion ads. I didn't add those yet. I think this is a great way to sift through articles that may or may not interest you. There's 40 different articles on display in the home page, and you can page through to see older ones.

![The redesigned Pony Foo home page][12]

This format made a lot of sense in search results, which used the same stupid _"render most of the article"_ "strategy" as the home page, and failed miserably to help you find what you were looking for.

![A search for the ponyfoo tag][13]

# Publication History

The [publication history](/articles/history "Repository of articles published on Pony Foo") _(ex. "archives")_ is probably the page that changed the least. For what it's worth it now contains a brief history of the site, and a count of the amount of articles published on the site. Can you guess the number?

> All the while I was working on the redesign I also had my eye on performance.

# Performance

One of the glaring issues in performance at Pony Foo was that I was serving up huge chunks of data even on pages that didn't need any of it. For instance, the home page served the introduction to six different articles, but the entire body of those articles was served as JSON for later use -- even though it wasn't being *actually used*. The same issue could be observed in search result pages and the publication history. The publication history made this particularly nasty, as that page contains every single article ever published on Pony Foo. That means that the page was receiving JSON models to render pretty much the entire site!

The other performance issue blocking this redesign was the fact that I was pulling critical CSS content for just the home page. That means that if the home page has different styles than some other content, that other content would be improperly styled while the non-critical CSS was being loaded. I fixed this by pulling critical CSS from many different pages and serving the inlined CSS according to the page being requested. This involved a bit of rework, and I'm planning an article on the subject as I think it's something most of us should be doing in our sites and apps -- it's **not** just for blogs.

# Easter Egg

Also, what do you think of the, uh, easter egg? I would show you, but then it wouldn't be an easter egg anymore.

> Hope you like the new design and that it makes for a more pleasant reading experience here on Pony Foo!

Any and all feedback is appreciated. If you want to open a discussion on some front, don't hesitate to email me at [hello@ponyfoo.com](mailto:hello@ponyfoo.com), and thanks!

  [1]: https://i.imgur.com/UomuKhN.png
  [2]: https://i.imgur.com/VCUxXGq.png
  [3]: https://i.imgur.com/0VXJ2OU.png
  [4]: https://i.imgur.com/upji3LC.png
  [5]: https://github.com/bevacqua/woofmark "bevacqua/woofmark on GitHub"
  [6]: http://creativecommons.org/licenses/by-nc/2.5/ "Creative Commons Attribution-NonCommercial 2.5 License."
  [7]: https://i.imgur.com/5faPo3U.png
  [8]: http://creativecommons.org/licenses/by-nc/2.5/ "Creative Commons Attribution-NonCommercial 2.5 License."
  [9]: https://i.imgur.com/DRl3Mxl.png
  [10]: https://i.imgur.com/1kCugfn.png
  [11]: https://i.imgur.com/IFE4bI5.png
  [12]: https://i.imgur.com/iv3T86s.png
  [13]: https://i.imgur.com/nlwv7IO.png
