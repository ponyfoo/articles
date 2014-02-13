# You don't need a TODO app

Recently, [a tip on Coderwall][1], about how to organize your TODO list, brought about [a discussion on Hacker News][2]. In this brief post, I'd like to provide my take on the subject.

![todo.jpg][3]

I've never been much of a fan of TODO lists, definitely _never used a TODO application_, and I'm one of those people who have no idea why they even exist, in the first place. Regardless, I've always kept these kinds of lists laying around in a variety of places, some more effective than others.

For instance, I've tried using _Sticky Notes_ in Windows, and Ubuntu, but these never stick with me (bad pun intended). I usually don't even look at them, or I close them, or hide them, it just doesn't do it for me. They're pretty, but they don't get the job done. I end up ditching them every time. You have to find what works well for you. **What kind of task-tracking is making you actually execute on those tasks?**


  [1]: https://coderwall.com/p/kpd7ra "The best todo list app for any developer"
  [2]: https://news.ycombinator.com/item?id=7162131 "Check out the comments on Hacker News"
  [3]: http://i.imgur.com/OvMZBs9.jpg

Sending mails to myself, and keeping them in my Gmail inbox, is something I've done often to remind myself of things I ought to do. I've always proudly kept my inbox below ~5 items, and if you can't do that _(because apparently it's really hard for people to [use filters in Gmail][1], or something)_, then sending mails to yourself might not be a very good idea. If you are one of the illuminated ones who can keep his inbox clean of spam, then you might want to try it. I also make it a point to create lots of labels and choosing the _"Show if unread"_ option. The single most important of the labels I use, is **Notes**. I use a filter which tags all my mails to self with that label.

I often use `TODO.md` files inside a project root, which usually contain a brief mention of where I left at the previous day, and what steps I was going to do next. Kind of a quick reminder of where I left off so I can quickly pick up the tab the next day. This is a **particularly useful thing to do on fridays** before leaving for home, and also mid-week, when you end the day in the middle the zone, with a bunch of things in your head, which you just know you're going to have _completely forgotten_ by the next morning.

### A `todo` bash function

I've always liked keeping a clean desktop, with absolutely no icons whatsoever. As such, I feel that the best way to keep a TODO list I will actually act upon, is to **have each item be a file in my desktop**. That way, I'd always feel the urge to delete the items as soon as possible. Somehow, deep down, I feel this makes me finish my tasks sooner, rather than later. I guess there's something psychology can explain about that. At the very least, it makes me add _relevant_ TODO items, as **I don't want junk laying around** in my precious desktop real estate.

Without further ado, behold my `todo` bash function.

```bash
# TODO
todo () {
  IFS_OLD="$IFS"
  IFS=$'\n'
  touch $(echo ~/Desktop/$@)
  IFS="$IFS_OLD"
}
```

`IFS` changes the [internal field separator][2] of the shell, and that trick allows my function to `touch` files with spaces on them.

Below is a list of usage examples, which will create empty files in my desktop named as the TODO items. Each of these will create a single file.

```shell
todo bugfix
todo feed the cat
todo "review pull requests"
```

### Gmail, `~/Desktop` and More

Combining all three of these, you get a pretty organized way of going through your day without spending time struggling with TODO apps getting in your way.

I use Gmail items to remind myself of tasks I don't have to get to immediately, but maybe at some point in the next month or two. Project level items, which lie in the project directory, change pretty much on a workday-to-workday basis, and contain exactly what I was working on and what's coming up. This is not to be confused with actual planning, which is done in a project management board of some sort: [Jira][3], [Pivotal][4], etc. Lastly, the `todo` bash function allows me to add a TODO item on my desktop from my command line, while sitting on any directory, to my desktop.

Below is a summary of the tasks I have laying around at the moment.

**Gmail Inbox + Notes label**

- a gh notification of an issue I have to resolve at some point
- an unresolved email about reviewing chapter 2 of my book
- a link to a camera I want to buy as a present
- some unread e-books

**~/Desktop**

- fix issues by reviewers and susan (book)
- update figures (book)

**Projects**

Clean at the moment, I don't have any unfinished tasks, or knowledge which I can't just pick up from the [Trello][5] board.

You can find this bash function, and related shell things, [in my dotfiles][6].

  [1]: https://support.google.com/mail/answer/6579?hl=en "Using Filters, Gmail Help"
  [2]: http://en.wikipedia.org/wiki/Internal_field_separator "Internal field separator, Wikipedia"
  [3]: https://www.atlassian.com/software/jira "Jira Project Management and Issue Tracking"
  [4]: https://www.pivotaltracker.com/ "Pivotal Tracker"
  [5]: https://trello.com/ "Trello Project Management"
  [6]: https://github.com/bevacqua/dotfiles/blob/0ba35a82c9d876a92e9841f96f6a2d5da5e345c1/zsh/aliases#L4-L10 "bevacqua/dotfiles aliases on GitHub"

[productivity tips gmail todo bash]
