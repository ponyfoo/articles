Progressive Web Apps have *several* requirements.

# Requirements

First off, you'll need `ServiceWorker` to be installed and activated to be able to make a website into a progressive app. That means you need to:

- Understand Promises ([article][1], [visualization][2])
- Implement secure conenctions ([using CloudFlare][3], [LetsEncrypt][4])
- Implement `ServiceWorker` ([demo][5], [tutorial][6], [article][7])

The upside is that most of these requirements are great things to do. Understanding promises, even if you don't particularly like them, is a necessity now that they're embedded into the language and being relied upon by web API interfaces such as `fetch` and most of the API related to `ServiceWorker`. Secure connections are now easier than ever with tools like CloudFlare and LetsEncrypt. Encryption is a necessity for any serious business, and a nice addition to "hobby" websites like this blog. Finally, `ServiceWorker` should be a no-brainer for every single website out there. The fine grained control over network requests, caching, and the ability to keep a site functional even when humans are offline are all immensely helpful.

Once you've checked off all the requirements mentioned above, implementing a progressive web app is super easy!

# Implementing Progressive Web Apps

The next thing you'll need, once you've completed the requirements, is to put together a `manifest.json`. Tutorials about the manifest format are scarce, but [the specification is simple enough][8] that you can browse it for things you want to configure -- such as the `name`, `orientation`, or the `display` options. Here's the one I'm using for Pony Foo.

```json
{
  "short_name": "Pony Foo",
  "name": "Pony Foo",
  "start_url": "/",
  "display": "standalone",
  "orientation": "landscape",
  "lang": "en",
  "icons": [{
    "src": "/apple-touch-icon-144x144-precomposed.png",
    "sizes": "144x144",
    "type": "image/png"
  }]
}
```

Once you've put together your `manifest.json`, you'll have to link to it from your pages, like so:

```html
<link rel='manifest' href='/manifest.json'>
```

In order to support older mobile browsers, you should also include the following meta tag.

```html
<meta name='mobile-web-app-capable' content='yes'>
```

You'll also need a `<link>` to an image of your site's icon. You might want to check out the writings on [article on touch icons][9] from Mathias Bynens, or maybe just [take a look at the icons I'm using][10] for this blog.

```html
<link rel='icon' sizes='192x192' href='icon-192x192.png'>
```

That's about it.

> Of course, you need to implement `ServiceWorker` first, and that's a pretty big leap, but given that `ServiceWorker` is a progressive technology, it's **completely risk-free to implement** _today_. You'll get the benefits in supporting browsers and those that don't support `ServiceWorker` will simply ignore your `ServiceWorker`. Eventually, those browsers will become up to date and support `ServiceWorker` too, and your site will immediately get ahead the pack in those cases as well!

[1]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
[2]: http://bevacqua.github.io/promisees/ "Promisees Visualization Tool"
[3]: /articles/securing-your-web-app-in-3-easy-steps "Securing Your Web App in 3 Easy Steps on Pony Foo"
[4]: https://letsencrypt.org/ "Let's Encrypt encrypts the web for free"
[5]: https://github.com/bevacqua/flickr-lightbox "bevacqua/flickr-lightbox on GitHub"
[6]: https://css-tricks.com/serviceworker-for-offline/ "Making a Simple Site Work Offline with ServiceWorker on CSS-Tricks"
[7]: /articles/serviceworker-revolution "ServiceWorker: Revolution of the Web Platform on Pony Foo"
[8]: https://w3c.github.io/manifest/ "Web App Manifest Specification"
[9]: https://mathiasbynens.be/notes/touch-icons#icons "Everything you always wanted to know about touch icons"
[10]: https://github.com/ponyfoo/ponyfoo/blob/b0b633f49c940611f806e751a2fb9876cde57a76/views/server/layout/layout.jade#L20-L29 "ponyfoo/ponyfoo on GitHub"
