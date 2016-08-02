In his post, [@ppk][1] argues something that I've heard before, that we would do good to consider pacing down on new developments -- and how we should focus on improving the most rudimentary things that are still so very impossible to work with on the web. While I don't think a moratorium, nor slowing down the pace are proper answers -- the problem is unquestionably there:

> How did we get to the point where we are able to have realtime bi-directional peer-to-peer communication between visitors to our site, yet we can't determine the styles for a measly `<select>` input, we can't customize form elements like `<input type='check'>`, we can't use pseudo-selectors like `:before` or `:after` on certain elements like `<img>` or `<input>`, because [these are "replaced elements"][2] -- _**what the hell is that?**_

To me, **the problem is not that we are moving too fast** on cutting-edge technologies such as WebRCT, HTTP 2.0, enforcing TLS, Service Workers, ES6, ES7, WebP, Web Audio, WebGL and _countless others_. Rather, <mark>the problem is that we are making no effort whatsoever to fix things that have been broken forever</mark>, like being able to give basic styling to form elements in a _non-over-engineered way_.

I vehemently agree that constraints drive innovation, as [@ppk][1] points out in his post. I think the web platform is constrained enough as is, though. However, I do get what ppk is trying to get out there -- we are worrying too much about these complicated things and missing out on the essential. For us to keep on focusing on the complicated problems we all know and love, we should deal with the inexcusable pain points of the web, such as how lousy form elements behave, how lame native form validation is _(where by the way, you can't style the validation errors consistently nor effectively)_, and how JavaScript ends up becoming the de-facto excuse for all of this _"well you can do it with jQuery"_, _"well, it works if you add some JavaScript sugar cones on top"_. Problem is, this is as rudimentary of an excuse as _"it works on my machine"_, and we all know by now that's <mark>a terrible excuse</mark>.

_Right?_

[1]: https://twitter.com/ppk
[2]: http://stackoverflow.com/q/8012297/389745
