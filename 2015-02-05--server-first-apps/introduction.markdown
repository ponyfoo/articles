In the article, Tom goes on to explain how applications rendered on the server-side are clumsy when it comes to responsiveness. This is a fair point, server-side rendered applications typically rely on the [PRG _(POST-Redirect-GET)_ pattern][1]. They have HTML `<form>`s, users `POST` some data, the server processes the request, responds with a redirect to another page, and the client `GET`s that other page. These are pretty much the basics of the web. What's worse, as Tom notes, is that as you start adding AJAX calls to this server-side rendered content, you are now a slave to both state _(what you initially pulled from the server)_ and behavior _(users clicking on things)_ when it comes to updating the UI. **That is the ultimate nightmare of a web developer.**

Client-side rendered apps, in contrast, can be _way faster than that_. Once the initial payload is downloaded, interpreted, and executed, client-side JavaScript can set up its own **smart caching on the client-side**, avoiding roundtrips to the server for data it already has, and it can also set up routing on the client-side to emulate incredibly-fast roundtrips. It can even have the server spewing information downstream as soon as it has any fresh data to offer, [using WebSockets][2]. The issue of updating the UI as a slave to many masters is long gone, since you just update the UI as the client-side JavaScript engine demands of you.

_And so the story goes..._

The answer to a productive and maintainable web development orientation, _that also favors customers_, doesn't lie in one or the other, but rather in _the combination of both approaches_. In this article, I'll explore what all of this means.

[1]: http://stackoverflow.com/questions/tagged/post-redirect-get
[2]: http://socket.io/
