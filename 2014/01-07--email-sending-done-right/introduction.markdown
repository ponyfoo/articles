> In the future, I'll abstain from tightly coupling major pieces of functionality together, and instead force myself to **write reusable components which are well documented**.

The blog repository grew in size considerably _while I was actively developing it_, but it became clear after a few months that some of the design decisions I had made were wrong, such as not doing any server-side rendering at all. I've set out to revamp the platform, and in doing so, I started to extract modules from the core repository.

One such piece is the emailing functionality. In this article I'll examine the design decisions behind [campaign][1], the comprehensive email library I extracted from Pony Foo, and detail what sets it apart from existing emailing packages.

Spoiler: **it's modularity**.

[1]: https://github.com/bevacqua/campaign
