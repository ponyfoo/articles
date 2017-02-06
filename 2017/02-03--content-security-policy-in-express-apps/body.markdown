# `Content-Security-Policy` in Express

If you're using Express, it's really simple to write maintainable CSP directives using [`helmet-csp`][helmet]. To implement the `img-src` rule we were talking about, we'd only have to write code link in the following snippet, and `helmet-csp` will take care of adding the appropriate header to our server's HTTP responses.

```js
const csp = require(`helmet-csp`)

app.use(csp({
  directives: {
    imgSrc: [`imgur.com`]
  }
}))
```

We could also use `'self'` to allow image resources from the same origin. Note that `'self'` must be in quotes.

```js
app.use(csp({
  directives: {
    imgSrc: [`'self'`, `imgur.com`]
  }
}))
```

There is a directive called `default-src` that's used as fallback for any undeclared directives. If we set `default-src: 'self'`, for example, we'd allow scripts, styles, and every other kind of resource to be loaded from the same origin.

```js
app.use(csp({
  directives: {
    defaultSrc: [`'self'`],
    imgSrc: [`'self'`, `imgur.com`]
  }
}))
```

Keep in mind that [`default-src`][default-src] **isn't inherited** by other declared rules, meaning that if we now removed `'self'` from [`img-src`][img-src], only images from `imgur.com` would be allowed.

There are **many** different directives we can set. What follows is a table [based off of MDN documentation][mdn], containing just the 15 most relevant rules. As you can see, we weren't exaggerating when we said CSP allows us to determine where resources can be loaded from **on a granular level**. We can make our CSP header as complicated as we want it to be, with directives ranging from simple rules such as where images should be allowed to load from with [`img-src`][img-src], to more severe ones like blocking all mixed content or restricting how the page works using the [`sandbox`][sandbox] directive.

Directive | Description
----------|------------------------------------------------------------------------------------
[`block-all-mixed-content`][block-all-mixed-content] | Prevents loading any assets using HTTP when the page is loaded using HTTPS.
[`child-src`][child-src] | Defines the valid sources for web workers and nested browsing contexts loaded using elements such as `<frame>` and `<iframe>`.
[`connect-src`][connect-src] | Restricts the URLs which can be loaded using script interfaces
[`default-src`][default-src] | Serves as a fallback for the other fetch directives.
[`font-src`][font-src] | Specifies valid sources for fonts loaded using `@font-face`.
[`frame-src`][frame-src] | Specifies valid sources for nested browsing contexts loading using elements such as `<frame>` and `<iframe>`.
[`img-src`][img-src] | Specifies valid sources of images and favicons.
[`media-src`][media-src] | Specifies valid sources for loading media using the `<audio>` and `<video>` elements.
[`object-src`][object-src] | Specifies valid sources for the `<object>`, `<embed>`, and `<applet>` elements.
[`report-uri`][report-uri] | Instructs the user agent to report attempts to violate the Content Security Policy. These violation reports consist of JSON documents sent via an HTTP POST request to the specified URI.
[`require-sri-for`][require-sri-for] | Requires the use of SRI for scripts or styles on the page.
[`sandbox`][sandbox] | Enables a sandbox for the requested resource similar to the `<iframe>` sandbox attribute.
[`script-src`][script-src] | Specifies valid sources for JavaScript.
[`style-src`][style-src] | Specifies valid sources for stylesheets.
[`upgrade-insecure-requests`][upgrade-insecure-requests] | Instructs user agents to treat all of a site's insecure URLs (those served over HTTP) as though they have been replaced with secure URLs (those served over HTTPS). This directive is intended for web sites with large numbers of insecure legacy URLs that need to be rewritten.

Granted, the above table can feel intimidating. To get started with CSP, however, we don't need to list every single rule. Generally speaking, a good idea is to set a `default-src` of `'self'` and then whitelist other origins to allow specific resources to load. Using `report-only` is a great way of finding out what those other resources could be.

# To get started, let's use `report-only`

The `Content-Security-Policy-Report-Only` header is identical to the `Content-Security-Policy` header, except that it behaves like a dry run. The policy won't be enforced, *-- resources will continue to load as they were --* but the configured `report-uri` will be requested with a `POST` message and a JSON payload.

I'll be using the [`winston`][winston] logger to persist log messages for the CSP report, but you can use anything you'd like. Odds are, you already have some sort of logging solution in place _-- use that!_

```js
const winston = require(`winston`)

app.use(csp({
  directives: {
    defaultSrc: [`'self'`],
    reportUri: `/api/csp/report`
  },
  reportOnly: true
}))

app.post(`/api/csp/report`, (req, res) => {
  winston.warn(`CSP header violation`, req.body[`csp-report`])
  res.status(204).end()
})
```

Once you set up CSP reporting, your server will start logging reports of every single resource that would've been blocked by your CSP header. As the days pass, you'll collect considerable amounts of data on the resources CSP would block.

You could fix the violations by relaxing your CSP directives, for example by adding origins you'll allow images to be loaded from; or by changing how your site is implemented so that the resources that would have been blocked by the CSP directive aren't requested anymore.

Keep adding directives to your CSP header without removing `reportOnly: true` from your `helmet-csp` configuration. As you tweak your header, you should see the amount of reports shrinking to a point where it'll be safe to remove the `reportOnly: true` option. At this point, your CSP header will be in effect and requests for resources from untrusted origins will be blocked.

Generally speaking, there's two hurdles to overcome when setting up a CSP header for production use.

Ads served by an ad network tend to load third party scripts, loading images, styles, and scripts from an assortment of different origins. Unless the ad network offers documentation on how to set up your CSP header to allow them to serve advertisements unimpeded, it can be hard to detect every origin casually in order to whitelist them. This is one of the reasons why the "report-only first" approach is recommended.

Inline scripts and inline styles can also be a hassle. Any occurrence of `onclick` handlers directly in your HTML code, inline script tags such as your typical Google Analytics snippet, or inline styles such as those recommended to optimize [critical rendering path performance][crp] will be blocked by CSP unless you add the shame-inducing `'unsafe-inline'` source to [`script-src`][script-src] and [`style-src`][style-src] directives, respectively. The same goes with`'unsafe-eval'`, which whitelists JavaScript code such as `` eval(`code`) ``, `` setTimeout(`code`) `` or `` new Function(`code`) ``._

# Inlining safely using a `nonce`

You probably have inline styles and scripts like the Google Analytics snippet we've mentioned above. To whitelist these snippets without turning on the `'unsafe-inline'` rule, we can use a nonce. A nonce is an unique code we generate for every request, using a module such as [`uuid`][uuid].

In the following piece of code, we generate a nonce for every request.

```js
const uuid = require(`uuid`)

app.use((req, res, next) => {
  res.locals.nonce = uuid.v4()
  next()
})
```

We then tell the CSP directive what the nonce for the current request is. Any inline scripts that don't have the same nonce in a `nonce` attribute will be blocked.

```js
const csp = require(`helmet-csp`)

app.use(csp({
  directives: {
    defaultSrc: [`'self'`],
    scriptSrc: [`'self'`, (req, res) => `'nonce-${ res.locals.nonce }'`]
  }
}))
```

Lastly, we apply the nonce as an attribute for any inline scripts and styles we have. This tells CSP-compliant browsers that the script is safe to execute.

```js
// in the real world, this would be in your view engine:
app.use((req, res) => {
  res.end(`<script nonce='${ res.locals.nonce }'>alert('whitelisted!')</script>`)
})
```

You can repeat the process for inline styles, but using [`style-src`][style-src] and `<style nonce='...'>`.

# Closing the loop with `upgrade-insecure-requests`

Using CSP's `upgrade-insecure-requests` directive, we could have the user agent automatically upgrade HTTP requests to HTTPS, effectively transforming code like this:

```xml
<img src='<mark>http://</mark>cats.com/hairy-cat.png' />
```

Into its secure counterpart:

```xml
<img src='<mark>https://</mark>cats.com/hairy-cat.png' />
```

Enabling this behavior in `helmet-csp` is merely a matter of setting `upgradeInsecureRequests: true`.

```js
app.use(csp({
  directives: {
    upgradeInsecureRequests: true
  }
}))
```

# CSP in Express, by Example

Here's an example of how we compose the CSP header for Pony Foo. There's quite a few whitelisted external services, and we weren't able to completely get rid of inline styles, but it's a good start!

```js
const csp = require(`helmet-csp`)
const uuid = require(`uuid`)
const env = require(`./env`)
const authority = env(`AUTHORITY`)
const authorityIsSecure = authority.startsWith(`https`)
const authorityProtocol = authorityIsSecure ? `https` : `http`

function generateNonce(req, res, next) {
  const rhyphen = /-/g
  res.locals.nonce = uuid.v4().replace(rhyphen, ``)
  next()
}

function getNonce (req, res) {
  return `'nonce-${ res.locals.nonce }'`
}

function getDirectives () {
  const self = `'self'`
  const unsafeInline = `'unsafe-inline'`
  const scripts = [
    `https://www.google-analytics.com/`,
    `https://maps.googleapis.com/`,
    `https://static.getclicky.com/`,
    `https://in.getclicky.com/`,
    `https://cdn.carbonads.com/`,
    `http://srv.carbonads.net/`,
    `${ authorityProtocol }://adn.fusionads.net/`,
    `https://platform.twitter.com/`,
    `https://assets.codepen.io/`,
    `https://cdn.syndication.twimg.com/`
  ]
  const styles = [
    `https://fonts.googleapis.com/`,
    `https://platform.twitter.com/`
  ]
  const fonts = [
    `https://fonts.gstatic.com/`
  ]
  const frames = [
    `https://www.youtube.com/`,
    `https://speakerdeck.com/`,
    `https://player.vimeo.com/`,
    `https://syndication.twitter.com/`,
    `https://codepen.io/`
  ]
  const images = [
    `https:`,
    `data:`
  ]
  const connect = [
    `https://api.github.com/`,
    `https://maps.googleapis.com/`
  ]

  return {
    defaultSrc: [self],
    scriptSrc: [self, getNonce, ...scripts],
    styleSrc: [self, unsafeInline, ...styles],
    fontSrc: [self, ...fonts],
    frameSrc: [self, ...frames],
    connectSrc: [self, ...connect],
    imgSrc: [self, ...images],
    objectSrc: [self],

    // breaks pdf in chrome:
    // https://bugs.chromium.org/p/chromium/issues/detail?id=413851
    // sandbox: [`allow-forms`, `allow-scripts`, `allow-same-origin`],

    upgradeInsecureRequests: authorityIsSecure,
    reportUri: `/api/csp/report`
  }
}

app.use(generateNonce)
app.use(csp({
  directives: getDirectives()
}))
```

# Further Reading

- [Subresource Integrity][sri] -- a better way of trusting resources served by our CDN
- [Content-Security-Policy.com][cspcom] -- a good go-to resource which summarizes a lot of advice around CSP
- [GitHub's CSP journey][gh-csp] -- where they explain the steps they took in implementing CSP
- [GitHub's post-CSP journey][gh-csp-p2] -- where they go further and dive into more advanced tactics to zero in on XSS attack mitigation

If you liked this article, consider subscribing to [Pony Foo Weekly][weekly], our periodic email newsletter on front-end development! 💌

[helmet]: https://github.com/helmetjs/csp "helmetjs/csp on GitHub"
[mdn]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#Directives "Content-Security-Policy Directives"
[block-all-mixed-content]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/block-all-mixed-content "Read the documentation about the 'block-all-mixed-content' directive on MDN."
[child-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/child-src "Read the documentation about the 'child-src' directive on MDN."
[connect-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/connect-src "Read the documentation about the 'connect-src' directive on MDN."
[default-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src "Read the documentation about the 'default-src' directive on MDN."
[font-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/font-src "Read the documentation about the 'font-src' directive on MDN."
[frame-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-src "Read the documentation about the 'frame-src' directive on MDN."
[img-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/img-src "Read the documentation about the 'img-src' directive on MDN."
[media-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/media-src "Read the documentation about the 'media-src' directive on MDN."
[object-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/object-src "Read the documentation about the 'object-src' directive on MDN."
[report-uri]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri "Read the documentation about the 'report-uri' directive on MDN."
[require-sri-for]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-sri-for "Read the documentation about the 'require-sri-for' directive on MDN."
[sandbox]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox "Read the documentation about the 'sandbox' directive on MDN."
[script-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src "Read the documentation about the 'script-src' directive on MDN."
[style-src]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src "Read the documentation about the 'style-src' directive on MDN."
[upgrade-insecure-requests]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests "Read the documentation about the 'upgrade-insecure-requests' directive on MDN."
[weekly]: /weekly "Pony Foo Weekly Email Newsletter"
[sri]: https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity "Subresource Integrity on MDN"
[gh-csp]: https://githubengineering.com/githubs-csp-journey/ "GitHub's CSP journey"
[gh-csp-p2]: https://githubengineering.com/githubs-post-csp-journey/ "GitHub's post-CSP journey"
[cspcom]: https://content-security-policy.com/ "Content Security Policy Quick Reference Guide"
[winston]: https://github.com/winstonjs/winston "winstonjs/winston on GitHub"
[crp]: /articles/critical-path-performance-optimization "Critical Path Performance Optimization at Pony Foo"
[uuid]: https://github.com/kelektiv/node-uuid "kelektiv/node-uuid on GitHub"
