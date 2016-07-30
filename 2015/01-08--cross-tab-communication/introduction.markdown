I wanted an elegant solution to the following scenario: suppose a human walks into your website, logs in, **opens a second tab**, and logs out in that tab. He's still _"logged in"_ on the first tab, except anything he touches will either redirect them to the login page or straight blow up in their face. A more inviting alternative would be to figure out that they're logged out and do something about it, such as display a dialog asking them to re-authenticate, or maybe the login view itself.

You could use the WebSocket API for this, but that'd be overkill. I wanted a lower-level technology flyswatter, so I started looking for cross-tab communication options. The first option that popped up was using cookies or `localStorage`, and then periodically checking whether they were logged in or not via `setInterval`. I wasn't satisfied with that answer because it would waste too many CPU cycles checking for something that might not ever come up. At that point I would've rather used a _["comet"][1] (also known as long-polling)_, Server-Sent Events, or WebSockets.

I was surprised to see that the answer was lying in front of my nose, it was `localStorage` all along!

[1]: http://stackoverflow.com/a/12855533/389745
