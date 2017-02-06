# What is `Content-Security-Policy`?

CSP is an HTTP header that helps you mitigate XSS risk by preventing resources from untrusted origins from loading. CSP comes with several different directives, each of which serves a specific purpose. For instance, the `img-src` directive is used when loading images, `script-src` is used when loading scripts, `connect-src` is used for XHR, `WebSocket` and friends, and so on.

Each CSP directive lets you indicate which origins are trusted by using a whitelist-based approach. User agents which support CSP will avoid fetching resources that don't match your server's CSP directives. This means our server can determine, at a granular level, which origins are allowed for which kinds of resources.

For instance, you might serve all of your images through the `imgur.com` service, and thus you could set the header to `Content-Security-Policy: "img-src: imgur.com;"` and prevent images from any origins other than `imgur.com` from loading. In a similar fashion you could limit script loading to just a subdomain of your site, preventing XSS attacks from loading scripts from `malicious.com` or from anywhere else you haven't proactively approved.
