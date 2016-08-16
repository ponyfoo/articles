There were several challenges that had to be sorted out before the implementation, which didn't take that long once I settled for an idea on how I wanted to approach it.

1. A script that converted any database article into its file system representation was necessary so that I could use that representation when talking to git.

1. The files need to be straightforward.

    - A huge JSON file doesn't make a lot of sense for Markdown articles, and a single Markdown file might prove confusing due to the amount of fields present on a Pony Foo article.
    - The ability to see the rendered article on GitHub is pretty important, but the problem is that the Markdown in Pony Foo has diverged quite a bit from GitHub's Markdown. At the same time, articles rely pretty heavily on domain-relative links _(such as [`/subscribe`][sub])_ and expecting the git user to enjoy an article by looking at half a dozen different files didn't seem reasonable.
    - Not every field needs to be on git. The git representation can act as a mirror of the database, which is the single source of truth. Which fields should be on git?

1. The repository needs to be provisioned with all pre-existing articles. I needed a script that went through every published article and converted it to files in a git repository.

1. Drafts need to be excluded from the repository. They haven't been published on the site yet, and as such providing them in the open in an unpolished form isn't the best idea. Since drafts aren't available on git, we don't need to concern ourselves with publishing or handling new articles being created directly through git.

1. Whenever an article is updated on the website, we need to update its files and push to git.

1. Whenever an article is updated on git, we need to update the web version.

Let's follow the logical progression. We'll start with provisioning the repository, and then look at how we can keep it up to date whenever an article is updated on the site. Once the repository is kept up to date with changes to the website, we can look at reacting to updates made against git.

Shall we?

[sub]: /subscribe "Subscribe to Pony Foo articles!"
