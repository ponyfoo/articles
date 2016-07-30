> This article was originally published on [CSS-Tricks.com][3]. Content below presents minor editorial improvements.

ServiceWorker is a progressive technology, and in this article I'll show you how to take a website and make it available offline for humans who are using a modern browser while leaving humans with unsupported browsers unaffected.

Today, ServiceWorker has browser support in Google Chrome, Opera, and in Firefox behind a configuration flag. Microsoft is [likely to work on it][1] soon, and there's no official word from Apple's Safari yet. Given that the [implementation status for ServiceWorker across browsers][2] is not great yet, it's a great opportunity to get ahead of the pack. You won't affect unsupported users and the ones that are supported are going to greatly appreciate it.  
The effectiveness of implementing ServiceWorker depends on your user-base as well, given that if most of your users are on Chrome then you won't have to worry about support that much.

Before getting started, I should point out a couple of things for you to take into account.

[1]: https://twitter.com/jacobrossi/status/608291251121618944 "@jacobrossi on Twitter"
[2]: https://jakearchibald.github.io/isserviceworkerready/#navigator.serviceworker "Is ServiceWorker Ready?"
[3]: https://css-tricks.com/serviceworker-for-offline/ "Making a Simple Site Work Offline with ServiceWorker"
