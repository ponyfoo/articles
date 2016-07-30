Sitting between humans and websites, ServiceWorker puts you in the driver's seat. When a ServiceWorker is installed, you'll be able to  deliver responses to requests from the service worker _(which lies on the client-side)_, without necessarily hitting the network.

ServiceWorker is the evolution of the AppCache manifest. AppCache has been bashed for years because of several issues -- that were [extensively documented][1] -- in a resonating _"AppCache sucks"_ roar across the blogosphere. Most of the issues in AppCache revolved around it being a simple-looking declarative interface to offline behavior that, behind the curtains, had **a complicated ruleset that wasn't simple nor intuitive.**

> The ApplicationCache spec is like an onion: it has many many layers, and as you peel through them you’ll be reduced to tears.  
> _-- [Jake Archibald][2]_

On the other hand, ServiceWorker offers a programmatic API that allows you to accomplish so much more than AppCache ever could. Instead of almost-too-magical rules around a simplistic declarative syntax, ServiceWorker relies on plain JavaScript code.

Even though it doesn't have perfect [browser support today][3], ServiceWorker is just the beginning. Once you implement ServiceWorker in your sites, you'll be able to provide offline functionality for your pages. Yes, even when the human doesn't have any network connectivity at all, **they'll be able to access your site while offline**, provided that they've visited at least once before. And that's _just the basic offering_. ServiceWorker also enables you to send **Push Notifications**, just like mobile apps do, and even when the website is running in the background; you also get to use **Background Sync**, allowing you to execute one-time or periodic updates of the content in your site while it's backgrounded; and **Add to Home Screen**, which does exactly what it sounds like, making your website feel almost like a native app.

[1]: http://alistapart.com/article/application-cache-is-a-douchebag "Application Cache is a Douchebag - alistapart.com"
[2]: https://twitter.com/jaffathecake "@jaffathecake on Twitter"
[3]: https://jakearchibald.github.io/isserviceworkerready/ "Is ServiceWorker Ready?"
