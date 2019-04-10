# Check out the PR locally

If you haven't done this before, you can just add the author's remote (provided they're using a fork, otherwise you can skip this step) like so:

```shell
$ git remote add nico https://github.com/nico/repo.git
```

Then we can fetch the PR branch, let's assume it's called `fix-merge-conflicts` and it was merging `branch-with-conflicts` into `master`, fixing the conflicts along the way:

```shell
$ git fetch nico fix-merge-conflicts
$ git checkout fix-merge-conflicts
```

# Assess the situation

The `git log` should indicate they've merged a couple branches. The commit hashes are the key bits here.

```shell
$ git log -1 HEAD
commit a9c9ed7gcc0a76670d86df4b732f1f219ccb48de (HEAD -> fix-merge-conflicts)
Merge: <mark>aecf730802</mark> <mark>ba9918dad5</mark>
Author: Bloodninja <bloodninja@usenet.org>
Date:   Mon Apr 8 12:32:38 2019 +0000

    Merge remote-tracking branch 'branch-with-conflicts'
```

And now we can verify that these indeed belong to each of the branches we care about:

```shell
$ git branch --contains=<mark>aecf730802</mark>
  master
* fix-merge-conflicts
$ git branch --contains=<mark>ba9918dad5</mark>
  branch-with-conflicts
* fix-merge-conflicts
```

## Now for the magic

```shell
git checkout -b what-were-they-thinking
git reset --hard <mark>aecf730802</mark>
git merge --no-ff <mark>ba9918dad5</mark> # this will conflict
git add .
git commit -m WIP
```

At this point, `what-were-they-thinking` is the version of the conflict resolution where you just committed things exactly as `git merge` left them, conflict markers and all.

And, presto, the following command lets you see exactly what they did.

```shell
git diff what-were-they-thinking fix-merge-conflicts
```

Obviously all the commit hashes, branch names, and the author are made up here, and you'll have to use the actual output you get in each command to feed that into the following command, but you already knew that so I'll just stop mansplaining and typing these words now!

Do you have any other quick and dirty tricks like this that would blow most people's minds? This one certainly blew my mind, although I can't claim credit for it. All credit goes to my brilliant co-worker [Joe Gallo][joe-twitter], who taught me this nifty trick and is a `git` mastermind! 🥰

[joe-twitter]: https://twitter.com/CrazyJoeGallo "@CrazyJoeGallo on Twitter"
