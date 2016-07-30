> This is _the second part_ of [a previous article][1], where I outlined how to measure performance and identify issues, how to enforce a [performance budget][2], and how to do those things _<mark>in every push to continuous integration</mark>_.

This time around, we'll focus on _finding fixes_ for some of the most common performance pitfalls of the web today. To that end, we'll **crawl the web stack**. Each section describes the problem space and provides _a possible solution_ you could implement. Here's an overview of what you'll find in this article.

* TCP connection optimizations
* HTTP and HTTP/2 improvements
* Caching and CDN usage
* HTML and server-side rendering
* Critical CSS inlining and other CSS techniques
* Font loading strategies
* Efficiency when serving images
* A couple of tips regarding your JavaScript

**Let's dive into it.** We'll start _at the very bottom of the stack_: the TCP transport layer.

[1]: /articles/talk-about-web-performance
[2]: http://timkadlec.com/2013/01/setting-a-performance-budget/
