When I found out I would be working on porting an old Python codebase to Node, I was slightly excited. These kinds of projects always give you more creative freedom than the ordinary code maintenance gig, and something about the challenge of rewriting other people's code makes it fun as hell.

The excitement significantly faded once I actually got a look at what we were going to work with. Legacy code can be nasty, but I've been programming for 15 years and only a couple of times had I seen something like this. The authors had created their own framework, and it was a perfect storm of anti-patterns: no separation of concerns, mixed spaces/tabs for indentation, multiple names for the same concept, variables being overwritten by the exact same data coming from a different yet nearly identical method, magic strings... this mess could only have been the product of a room full of babbling monkeys copying code randomly from Google.

And yet, it was not the code's dismal quality that piqued my interest and led me to write this article. What I discovered after some months working there, was that the authors were actually an experienced group of senior developers with good technical skills. What could lead a team of competent developers to produce and actually deliver something like this? What I've come up is a list. **These are some bad habits that even experienced teams can get into which will severely affect your end product, more than any static code checker or development methodology could rescue it from.**

### Giving excessive importance to estimates

An important component of this project was the focus on deadlines, even to the detriment of code quality. If your developers have to focus on delivering rather than on writing good code, they will eventually have to compensate to make you happy. The two ways in which they can do this are over-estimating and over-promising, and they both come with added baggage.

Typically it's more difficult to over-estimate because the effects are immediately evidenced on project cost, so **your developers might choose to over-promise instead, and then skip important work such as thinking about architectural problems, or how to automate tasks, in order to meet an unrealistic deadline**. These tasks are often seen as added value, so they get cut without notice. Product quality will decline as you accumulate technical debt, and you'll find out about it later than you'd really want to, because reorganizing code later in a project costs exponentially more.

As an example, on this project I would find code that was obviously duplicated elsewhere, but it seemed that people were in such a rush to deliver that some developers would not bother to check if someone else had written the same method or SQL query before.

Sometimes estimates can even be deceiving. For example, Agile has a term called "velocity". The idea is to calculate how fast your team can deliver, and make the necessary changes to go faster. The problem is that it's not possible to come up with an accurate velocity in the short to mid term. The law of averages says that you can't look at past performance to gauge how fast you can go right now, because past performance is not a good indicator of future results.

> The law of averages is a layman’s term for a belief that the statistical distribution of outcomes among members of a small sample must reflect the distribution of outcomes across the population as a whole.

In truth, adeveloper can write a large amount of code one day, and she can take three days to write five lines of code after reading documentation and collaborating with teammates. **Averaging estimates will not net you valuable information in the short or mid term.**

## Giving no importance to project knowledge

As your project progresses, your team learns about the business, the concepts behind it and how they connect together. They also learn about the implementation as they write code, because you can never fully visualize how things will turn out and which challenges you will face. Some business fields even have an inherent complexity that takes a long time to digest.

As this was a full rewrite of old code, it was particularly interesting in this regard, because it serves to show whether your team's management understands project knowledge and how it impacts development. **If you're in a large project and there are modules for which you have no expert, no-one to ask about, that's a big red flag. ** The value in rewriting code rests entirely on taking advantage of what you learned the first time around, so value that knowledge dearly.

If you put a different team to do the rewriting, as was done in my case, you are ignoring all your learning and relying solely on your new team's skills, which likely won't make up for the lack of information. **A mediocre developer is going to do a better job at rewriting something he wrote himself, and he'll do it faster, than if you give the job to someone else entirely.**

Even hiring is impacted by project knowledge. The more information the project carriers, the longer it will take to bring people up to speed, so not only is domain knowledge more valuable then, it's also important to focus on hiring good people. If you don’t hire well, you’ll get tied up to a bad team that will go nowhere for months.

## Focusing on poor metrics such as “issues closed” or “commits per day”

> When a measure becomes a target, it ceases to be a good measure. - Goodhart’s law

At some point while I was getting up to speed on this project, somebody asked me why another developer was closing issues much faster than me, as if delivering faster is a good thing. As you can imagine, it took me a glance at his code to find four bugs in a single line. **Focusing on unreliable metrics such as this will completely derail your project, and cause people as much stress as deadlines.**

One metric that few seem to focus on is regression rate of issues. There are bugs such as null pointer exceptions that might show up much later, and if you're not tracking regressions, it will seem as if bugs keep springing up out of nowhere. In this situation, you'll never find the source of your problems, because you're looking in the wrong place.

Ultimately what matters is what is delivered to the client, how happy they are with the product, and how it affects their bottom line, but **it takes a lot of self-control to focus on delivered quality and ignore juicy metrics such as commit rate or issues closed.**

A good way to know if a metric is useful or not, is to try to understand what personal values it outlines. Concentrate on metrics that advertise good attention to details, good communication skills and good attitude, especially if they require great effort to cheat.

## Assuming that good process fixes bad people

Good process is portrayed as a kind of silver bullet in business. In my experience some companies, especially large ones with a poor hiring methodology, end up making their process increasingly strict to put toxic people in line, in turn restricting the creative freedom of the ones carrying the team. Not only that, but you still need people to carry out your process properly in the first place for it to work.

It never ends, and this entire problem can be disregarded by just fixing the hiring. **Talent makes up for any other inefficiency in your team. That's the entire point of favoring smart work over hard work.**

Developers can be especially tricky to communicate with. In such a complex codebase, I had to constantly ask others for help, and would not often have people happily set time aside to give me a hand. That does not reflect a good attitude, and in tough tasks things can get especially stressful if you have to ask for help and can only count on a few of your teammates to have both the knowledge and the will to give you a hand.

**You need people who can apply common sense and good taste, and who will call you out if your process is holding them back.** Every build tool, every static checker and every communication tool has good and bad use cases, and you need people to tell you about it, not to blindly apply something that looked good in a different context months ago.

## Ignoring proven practices such as code reviews and unit testing

Staying up to date on modern software development processes might not be enough to put a derailed project back on it's tracks, but it's certainly necessary anyway if you want your team to stay competitive. This is where proven practices step in, and there are a few of them. [Test-driven development has been shown to reduce defect rates](http://research.microsoft.com/en-us/news/features/nagappan-100609.aspx) 40% to 90%, with an increase in development time in the 15%-35% range. [Code reviews have also been shown to decrease defect rates](https://blog.codinghorror.com/code-reviews-just-do-it/), in some cases up to 80% over manual testing.

Imagine my dismay when I had to collaborate with a colleague on that legacy project and his screen displayed Notepad in its full glory. Using "search" to find methods might have been rad back in the nineties, but these days, **refraining from using tools such as modern IDEs, version control and code inspection will set you back tremendously.** They are now  absolutely required for projects of any size.

For an in-depth look at what is proven to work in software development, check out the book [Making Software: What Really Works, and Why We Believe It](https://www.amazon.com/Making-Software-Really-Works-Believe/dp/0596808321/ref=as_li_ss_tl?ie=UTF8&linkCode=ll1&tag=chrimaiospo06-20&linkId=84b7d672a6bbc66f55547d8208526ef2). It has a nice list of valuable and proven practices which have been backed by studies over the years.

## Hiring developers with no “people” skills

It's not that developers can't talk to other people. I was once a shy developer myself and eventually managed to stand in front of an audience and give a talk just fine.

The problem comes when someone isn't willing to even try, or becomes annoyed by requests to improve communication. If there's one thing that can speed up development time more than anything else that I've mentioned, it's improving communication. Particularly when you reduce distances and work on informational proximity, you enable a more fervent and articulate connection in the workplace. **It doesn't matter that the other guy might be ten thousand miles away. One Skype call can turn a long coding marathon into a five-minute fix.**

## Conclusions

When you enable and encourage working smart by using the best tools, proven techniques and great communication, software development will definitely flow more naturally. **What you can't assume is that just because you've signed up to apply Agile or some other tool, that nothing else matters and things will sort themselves out.** There's a synergistic effect in action here which can make a team exponentially more productive if they're set up right, and terribly slow and sloppy when no attention is paid to the details.
