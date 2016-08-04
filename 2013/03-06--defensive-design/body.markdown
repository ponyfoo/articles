[![defensive-design.png][1]](http://amzn.to/2aoGDKr "Defensive Design for the Web")

### Put yourself in the user's shoes

Some of the things they mention have to do with how users react to the **UX** you provide. Don't overwhelm your users with more information than they need. Realize _your users don't know the internals of your application_. They don't know nor care _exactly why something went wrong_, they care about how to solve that. Be human, **technical blabber will drive them away in droves**.

> Don't scare your users away from your site by telling them something like "Something went terribly wrong", try and be helpful instead, mention how to work around the issue, or provide an alternative, or prompt them to try again later. Don't just tell them that _your site blew up_. They'll leave and **never come back again**.

![ysod.gif][2]

As a rule of thumb, tell your users _what went wrong_, but be as general as possible, while still being helpful. The only exception to this should be errors as the result of _ill intended behavior_, such as manipulating AJAX requests to your site. In those cases, it's fine to be a _little more obtuse_.

### As a Security Concern

Being too helpful about exactly what went wrong and how it happened, can go a long way in helping a malicious user gain access to your servers. You should never serve end users with detailed error descriptions, stack traces, server logs, or the like. But I guess, this again, is just common sense. Who's going to serve a full stack trace to the end users of a website, _right?_

### Be nice and generic, yet _concise_ and _consistent_

Make an effort to display error messages in **a consistent way** across your entire experience. Keep your error handling logic in _one place_, and it should be easy for your whole application to handle and display error messages in the same way. Not doing so certainly won't help your user in thinking your brand is unified and working as a unit.

If each independent submodule of your application displays error messages in a different way, or at _noticeably different levels of detail_, users will have a hard time feeling your product isn't merely a _bunch of modules mashed together_, and probably an unstable mess which will crumble in the presence of the smallest hiccup.

If your error messages are **consistent**, this will portray a message of _unity and cohesion_ which will go a long way towards **building trust** towards your application.

Obviously that's not the _only_ factor in building trust, but honest error messaging is _a start_.

**Be concise**. Tell your users what the problem is, and how to fix it.

> "Try again" just won't cut it, the user will _become frustrated_ and not know what you mean by that. Did you mean for them their login credentials were invalid? Did you mean the servers are overloaded right now and they should try again later? Did you mean the username they picked is already in use? **Be specific**.

![frustrated.png][3]

Use **meaningful** messages. "Invalid credentials", "Username in use. Pick another one". Explain to your user how to _get out of this situation gracefully_.

**Don't _inflict panic_ upon your users**.

### About that...

The book stresses that you should convey error messages in _red **bold** text on yellow backgrounds_. I disagree on this count. While you should certainly make your error messages _stand out_, you shouldn't make them scary, the user should feel that you are _merely informing_ them about something that _can happen on ocasion_, rather than sounding so alarmed about it.

> **OMG!** YOU MADE A _TERRIBLE_ MISTAKE!

The psychological impact of such error messaging is **negative**, you should try to convey your message as a _warning_, or just _a notice_, rather than an error that can't be salvaged. Also, **never put the blame on the user**. _Admit mistakes_ when internal errors turn up, **don't tell lame excuses**.

> Embrace your flaws and your users will embrace you.

In doing so, you are telling your users _you care about them_, people like that.

  [1]: https://i.imgur.com/JxkQ02e.png
  [2]: https://i.imgur.com/CGA63nm.gif "ASP.NET Yellow Screen of Death"
  [3]: https://i.imgur.com/CQVskxL.png "Don't frustrate your users"
