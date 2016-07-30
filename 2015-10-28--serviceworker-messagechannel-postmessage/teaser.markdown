Last week I wrote an article about [a caching strategy for progressive networking][1] that uses a cache first and then goes to the networking, sharing messages between web pages and a `ServiceWorker` to coordinate updates to cached content. Today I'll describe the inner workings of the [`swivel`][2] library that's used to simplify message passing for ServiceWorkers.

[1]: /articles/progressive-networking-serviceworker "ServiceWorker and Progressive Networking on Pony Foo"
[2]: https://github.com/bevacqua/swivel "bevacqua/swivel on GitHub"
