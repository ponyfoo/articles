# Squashing Commits

One of the first things I've learned when I started doing open-source was to squash my commits. Open-source folk like their commit logs clean and readable, a searchable and meaningful history of their repository. Squashing means turning a lump of commits into a single commit. This is **one of the things almost every maintainer out there asks you to do** before merging your PR, and you can quickly understand _why_.

Suppose you've created two commits and submitted a <a aria-label='pull request'>PR</a> against a repo you like, closing a bug you've filed a few hours earlier. The maintainer does some code review, asks you for a few changes, you commit some more. Now the PR has 5 commits, some of which undo work you did in the first few commits. Merging this PR as-is would **add a ton of different commits** for only <mark>one unit of work</mark> -- that is, fixing a single bug. Squashing the commits together improves readability of the changelog, and linkability of commits when referring to an individual bug fix.

To squash your latest commits, suppose there's 5 of them, you can run the following commands:

```shell
git reset HEAD~5
git add .
git commit -am "Here's the bug fix that closes #28"
git push --force
```

After running `git reset`, all your work will be unstaged, so you can then create another commit. This will rewrite history in the branch for your PR, but that's okay because nobody depends on that until it gets merged. And when that happens, only one commit will be pulled by everyone else!

# Amending your commits

If you want to change your latest commit before pushing -- <mark>because your commit message was a placeholder like _"foo"_</mark> -- you can use `--amend`, like so:

```shell
git commit --amend -m "Some other commit message."
```

If you had already pushed, you can still ammend the commit, but then you'll have to force-push. Again, **beware**, this rewrites history in `git`, so it's best to avoid force-pushing when possible.

```shell
git push --force
```

In line with squashing and amending commits, we should also take care of _thoughtful commit messages_.

# Commit Messages

The message in a commit should be descriptive enough to summarize the entire changeset. A good commit message might be `"fixed #18, where a dialog box would pop up in the wrong place"`, not so long that it becomes distracting, and not **so short** that it doesn't tell you anything about <mark>_what was changed_</mark>.

Something like `"foo"` is a terrible commit message. At that point, you might as well just create a commit with an empty message. This might be useful in those situations where you just want to redeploy to trigger another build in a CI system, without visiting their web interface.

```shell
git commit --allow-empty
```

Let's move on. What makes a good issue or pull request?

# Issues and Pull Requests

When it comes to creating an issue, you'll find that maintainers are much more welcoming if you've put any effort beforehand. That entails:

- **Reading documentation** about their library
- Figuring out if the issue has been _already reported_
- Taking the time to isolate the issue and provide <mark>precise bug reports</mark>
- Providing screenshots or reproduction steps

If you do at least one of the above, your reports will be taken more seriously and probably addressed more swiftly. If you don't _"have the time"_ to do research around the issue you're reporting, why would maintainers spend theirs fixing your issues with their library?

While yes, it's in their best interest to fix broken things, it's also nice to see contributors who actually go through the motions of at least trying to make things easier for the maintainers.

When it comes to pull requests, **try and match the code style** used in the repository you're contributing to. If they have a `CONTRIBUTING.md` file, that'd be a good place to learn about how they want you to approach contributions. Matching their coding style _will save you a lot of time_ in back and forth, git commits, and amendments. Plus, maintainers will be grateful for it!

Above all, <mark>**be nice**</mark>. Most of us do open-source just because we like the _"sharing mentality"_ in OSS, because we value the effort that others invest in open-source, or simply because we enjoy it. For most people however, it's not their day job, but something they do on their spare time. Even when it is, _they don't owe you anything_, so **don't make demands**.

> Ask nicely, and try to be helpful!

<mark>Have any questions or thoughts you'd like me to write about?</mark> _Send an email to [thoughts@ponyfoo.com][1]._ Remember to **subscribe** if you got this far!

[1]: mailto:thoughts@ponyfoo.com "Send me your questions and feedback!"
