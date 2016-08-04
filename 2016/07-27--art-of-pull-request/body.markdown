# Contributing Guidelines

Code review processes can be wildly different across open-source projects. The best place to start is to look for a [`CONTRIBUTING`][contrib] file in the top level directory for the repository. Large open-source projects usually have one of these. In it, maintainers lay out the *rules for engagement* when it comes to reporting issues, requesting features, asking for support, or contributing code to the repository.

While reading through a `CONTRIBUTING` file might sound like a waste of time, it's actually *going to save you time in the long run*. If you start contributing code to a repository right away, without going through their guidelines, you may be wasting time away by not conforming to their conventions. If you file an issue with insufficient information, maintainers are going to have to ask for more details, point you to the `CONTRIBUTING` file so that you actually read it, and generally waste time. At the same time, you'll be wasting your own time by having to go in repeatedly and update bits of your bug report every time.

Since these types of guidelines vary quite a bit from organization to organization _— and sometimes even from repository to repository under the same organization —_ we'll focus on the contributing guidelines for Kibana as an example we can learn and extract conclusions from.

The Kibana guidelines start with a table of contents. Right off the bat, you get a rough idea of what's covered -- it would seem there are instructions on how to report issues, answers to common questions, and a detailed walkthrough explanation on how to contribute code to the Kibana repository.

[![Kibana contributing guidelines][kbn]][contrib]

Good contributing guidelines contain helpful information such as how to set up the development environment, so that contributors don't have to figure things out on their own by poking in the dark. There's usually tons of information on how to run tests, how to lint, and how to build. In the Kibana case there's also helpful information around troubleshooting, such as fixing problems with SSL in development or fine-tuning source maps.

There's a couple of sections in the Kibana `CONTRIBUTING` guidelines which are a bit different than how other projects' guidelines are laid out. Firstly, there's how we assign priority to different issues. There's five different priority buckets where we place issues in. Priority is important to us, because we may want to fix high priority issues quicker, while lower priority issues don't require immediate attention.

> At any given time the Kibana team at Elastic is working on dozens of features and enhancements, both for Kibana itself and for a few other projects at Elastic. When you file an issue, we'll take the time to digest it, consider solutions, and weigh its applicability to both the Kibana user base at large and the long-term vision for the project. Once we've completed that process, we will assign the issue a priority.
>
> - **P1**: A high-priority issue that affects virtually all Kibana users. Bugs that would cause incorrect results, security issues and features that would vastly improve the user experience for everyone. Work arounds for P1s generally don't exist without a code change.
> - **P2**: A broadly applicable, high visibility, issue that enhances the usability of Kibana for a majority users.
> - **P3**: Nice-to-have bug fixes or functionality. Work arounds for P3 items generally exist.
> - **P4**: Niche and special interest issues that may not fit our core goals. We would take a high quality pull for this if implemented in such a way that it does not meaningfully impact other functionality or existing code. Issues may also be labeled P4 if they would be better implemented in Elasticsearch.
> - **P5**: Highly niche or in opposition to our core goals. Should usually be closed. This doesn't mean we wouldn't take a pull for it, but if someone really wanted this they would be better off working on a plugin. The Kibana team will usually not work on P5 issues but may be willing to assist plugin developers on IRC.

The other aspect of our contributing guidelines that's unique is the code review process.

# Reviewing Pull Requests _--- In Kibana_

Code review is almost always performed by a couple _(or more)_ of Kibana core engineers, but it's important to have our review process out there so that there are no surprises. We follow these guidelines both when creating a Pull Request ourselves, as well as when someone external to the organization submits a PR. Having a single process for everyone results in better quality and more consistency across all Pull Requests. Sometimes this can be a problem when you're trying to be friendly with an external PR, as you may feel inclined to lower the bar just for that one PR so that the contributor is happier, but being clear on the rules benefits everyone in the end.

> So, you've been assigned a pull to review. What's that look like?

The first rule is a nudge in the direction of the pull requester. Their contributions are important to them, and potentially to others, so don't stand around doing nothing. If you're assigned a PR, review it. Along the same lines, if the PR has several issues, "fail fast" doesn't really help here. Instead, point out all the different things that may be wrong with the PR, so that the contributor can fix them all at once instead of engaging in a ping pong match. You are lowering the chances the contributor will return the ball with every pong you serve, as they may become frustrated by all the waiting time, this is something you decidedly want to avoid.

> 1\. **Understand the issue** that is being fixed, or the feature being added. Check the description on the pull, and check out the related issue. If you don't understand something, ask the submitter for clarification.

As a contributor, this means an ideal PR links to the issue or issues it's trying to solve, have a description -- maybe including a couple of screenshots -- and contain a few notes in the commits.

For large commits, you should use the message as a summary and then go into detail in the commit message body. You can find great examples of this all over big open-source repositories like [`elastic/kibana`][kb] or [`elastic/elasticsearch`][es]. I also like tagging each commit with the affected aspect of the code base, such as `[test] Added a couple of tests to KbnServer.` or `[docs] Improved Contributing Guidelines.`.

> 2\. **Reproduce the bug** (or the lack of feature I guess?) in the destination branch, usually `master`. The referenced issue will help you here. If you're unable to reproduce the issue, contact the issue submitter for clarification.

Not a ton of value to be derived here for the contributor. If you're submitting a new feature, ensure that you're contributing something that doesn't already exist in the codebase _-- and that it wasn't being worked on either!_ If you're submitting a bug fix, ensure that the bug was present before your PR and that your PR fixes the aforementioned issues.

> 3\. **Check out the pull** and test it. Is the issue fixed? Does it have nasty side effects? Try to create suspect inputs. If it operates on the value of a field try things like: strings (including an empty string), null, numbers, dates. Try to think of edge cases that might break the code.

We don't have a ton of QA folks, so we rely on engineers to keep the code quality flag flying high. At the most crude level, we should figure out whether the changes have the desired impact -- do they fix the bug? Is the new feature working as expected? There should also be a decent bit of manual testing in order to ensure that the application isn't broken as a result of merging the PR. Even though the code reviewer is expected to perform these manual tests, there is no point in submitting a PR without performing at least some manual testing beforehand. The back and forth just spends time, a precarious currency you shouldn't be wasting! ⏳

> 4\. **Merge the target branch**. It is possible that tests or the linter have been updated in the target branch since the pull was submitted. Merging the pull could cause core to start failing.

Always test against the latest version of the upstream repository. In the case of the submitter, this also means that before submitting your PR, you should rebase against `master` *(or whatever branch you intend your PR to be merged into)*.

> 5\. **Read the code**. Understanding the changes will help you find additional things to test. Contact the submitter if you don't understand something.

Naturally, a code review wouldn't be complete if the reviewer didn't read the code. Being thorough is important, but as a reviewer you should keep in mind that not everyone will write code exactly in the same way as you do. That's okay, because you don't *own* the code base. Being able to live with different coding styles, even if they all fit into the same coding style guide, is an important aspect of reviewing a PR.

> 6\. **Go line-by-line**. Are there [style guide](https://github.com/elastic/kibana/blob/master/STYLEGUIDE.md) violations? Strangely named variables? Magic numbers? Do the abstractions make sense to you? Are things arranged in a testable way?

As a committer, you should try and abide by the rules set forth in contributing style guide. You can mostly get away with not breaking with the style you find in the files you're changing, or copying those coding styles when creating new files. A reviewer may opt to present an alternative way of architecting a piece of code if they anticipate it being counterintuitive or hard to follow in the future, when the PR is merged and away from carefully reviewing eyes. In the long run this may avoid code duplication and all kinds of confusion, so this step is crucially important to get right!

> 7\. **Speaking of tests** Are they there? If a new function was added does it have tests? Do the tests, well, TEST anything? Do they just run the function or do they properly check the output?

This should be a given when submitting code to a professionally maintained business project: write tests. Tests will be expected. You should write tests covering any changes you made. Sometimes you can get away with no tests, in cases when you fixed a bug where tests were already failing.

More often than not, you'll have to write new tests. In the case of a bug fix, introduce a test that would fail without your code changes, but passes after your code is applied. In the case of a new feature, write tests covering the new surface area in any public interfaces between your new feature and the rest of the code base. In this sense, an interface is anything that reaches into your code from the outside, or where your code reaches into external code.

> 8\. **Suggest improvements** If there are changes needed, be explicit, comment on the lines in the code that you'd like changed. You might consider suggesting fixes. If you can't identify the problem, animated screenshots can help the review understand what's going on.

Suppose for a minute that contributors are taco trucks. If we consider an open-source project as a voracious taco-eater, then it's paramount to the open-source project for taco trucks to be around. It's hard to keep the taco trucks lining up all around you, so it might be a good idea to nudge them your way and keep them coming back, because you're such a good customer.

To keep the taco trucks coming, you need to be gentle with them when purchasing their produce. This is particularly true if this is the first time you're buying from said taco truck!

All this is to say that, in order to keep contributors submitting code and not being frustrated, you need to be helpful and indicate what is and what isn't working about a PR. If you want something fixed in the PR, be gentle about it. Chances are, it's one of their first contributions and if given the chance they'll become better and better in adhering to your conventions over time -- especially if you're nice to them!

Not much to be said on the contributor side here, except it's nice to think of yourself as a taco truck! 🌮

> 9\. **Hand it back** If you found issues, re-assign the submitter to the pull to address them. Repeat until mergable.

Again, the back and forth everyone knows and hates. On both sides, try to keep the noise to a minimum. Some tips here were already addressed, such as having a reviewer submit as much feedback as possible every round, as to avoid friction. When it comes to contributing, try and anticipate the reviewers needs and wants. You may have a reason for not having implemented tests yet, thus maybe a TODO progress list may be in order, helping you communicate what parts of your PR are still missing. Adhering to conventions early and often will also help reduce friction and tighten the feedback loop.

> 10\. **Hand it off** If you're the first reviewer and everything looks good but the changes are more than a few lines, hand the pull to someone else to take a second look. Again, try to find the right person to assign it to.

The first reviewer may be unfamiliar with the affected portion of the codebase, or miss important aspects of a PR. Some times, bugs can be easy to miss in large changesets. Design problems can escape the first reviewer, too.

It's good to loop in a second reviewer, and maybe a third one too, and repeat the process. *Continuous integration tests should be passing by now.*

Hopefully the extra pairs of eyeballs will catch anything the first pair missed, and everybody will win. In any case, the bulk of the code review will typically be done by the first reviewer. As a general rule, though, the job of a contributor is to satisfy the reviewers. I see it as a game, where I need to pass a few levels before I complete my mission and eat some precious tacos. 🌮

> 11\. **Merge the code** When everything looks good, merge into the target branch. Check the labels on the pull to see if backporting is required, and perform the backport if so.

*The rewardingly boring parts!*

When enough reviewers gave a contribution their thumbs up 👍, the PR is ready for merge. Backporting may be necessary for bug fixes for issues that were already present in older versions of the codebase.

> Thank you so much for reading our guidelines! 🎉

It pays to be nice! 😅

# Getting Started

Generally speaking, you can always start contributing to a code base right away by reading through and improving their documentation. Once you feel a bit more comfortable with the repository, you can start picking away at issues. 

In the case of Kibana, you could grab a [<kbd>low fruit</kbd>][low] or [<kbd>P3</kbd>][p3] issue. Another effective way of detecting high-yield issues is sorting by "Most commented" or "Most 👍 reactions". Once you feel comfortable with our Pull Request and Code Review processes, nothing should stop you from grabbing a [<kbd>P1</kbd>][p1]. Keep in mind, though, to check that nobody else is working on those same issues yet!

Happy Pull Requesting!

[contrib]: https://github.com/elastic/kibana/blob/365cc821c766c044515ec80d70983c10bffafe9e/CONTRIBUTING.md "Contributing guidelines for Kibana on GitHub"
[kbn]: https://i.imgur.com/6oehAtQ.png
[kb]: https://github.com/elastic/kibana/commit/27ea119a9c6b4a0331ac78a56c5166e012c465cc "Sample commit for Kibana"
[es]: https://github.com/elastic/elasticsearch/commit/119026b4fb645a7cccd0df64ccd9e91b8285827b "Sample commit for Elasticsearch"
[low]: https://github.com/elastic/kibana/issues?q=is%3Aopen+is%3Aissue+label%3A%22low+fruit%22 "Low hanging fruit issues on Kibana"
[p3]: https://github.com/elastic/kibana/issues?q=is%3Aopen+is%3Aissue+label%3AP3 "P3 issues on Kibana"
[p1]: https://github.com/elastic/kibana/issues?q=is%3Aopen+is%3Aissue+label%3AP1 "P1 issues on Kibana"
