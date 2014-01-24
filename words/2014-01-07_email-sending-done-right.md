# Email Sending Done Right

[A week ago I wrote][1] about a few goals I've set for myself in 2014. In particular, I alluded to writing code that's more modular than what I've been writing so far:

> In the future, I'll abstain from tightly coupling major pieces of functionality together, and instead force myself to **write reusable components which are well documented**.

The blog repository grew in size considerably _while I was actively developing it_, but it became clear after a few months that some of the design decisions I had made were wrong, such as not doing any server-side rendering at all. I've set out to revamp the platform, and in doing so, I started to extract modules from the core repository.

One such piece is the emailing functionality. In this article I'll examine the design decisions behind [campaign][2], the comprehensive email library I extracted from Pony Foo, and detail what sets it apart from existing emailing packages.

Spoiler: **it's modularity**.

  [1]: /2014/01/01/a-year-in-review "A Year in Review"
  [2]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"

Sending emails often involves tons of overhead, you usually want to set up a few different things.

- Authentication credentials to either an email account or an email-sending service
- A pretty email layout so that your messages don't look like spam
- Templates for each of the different types of email you would send
- The ability to send out multiple emails using a single API call
- Debugging facilities for development
- Some way to embed images so you don't have to serve them

There already are popular solutions that deal with some of these things. For example, [Nodemailer][1] deals with the email sending part, but it doesn't provide any _easy to find_ examples of how to deal with a transactional API such as [Postmark][2] or [Mandrill][3].

Then there's [node-email-templates][4], which deals with templating, but is really opinionated about it. You are to use [`ejs`][5], you need to keep your templates in a directory, they need to be in such-and-such folder structure. Utter non-sense. The funniest part is that the library doesn't really do a lot, other than complicate your templating process and providing a few layouts for you to copy and paste.

# Enter [`campaign`][6]

[![campaign.png][7]][8]

When building out the project the first time around, I wrote a bunch of logic to be able to deal with email sending easily. I used [Mandrill][9] for a simple transactional email API, [Mustache][10] allowed me to write the templates, and I spent a lot of time fiddling with MailChimp's [responsive email templates][11], getting the layout just right. I also spent time dealing with embedded images, unsubcribe links, and Mandrill's [merge tags][12], used to customize emails sent to a lot of people at once.

I spent some time improving the API so that it wouldn't be all over the place, making it easy to interact with while keeping the layout, the views, and the models completely separated. This is what it looks like:

![email.png][13]

Now, I've extracted the logic from [Pony Foo's codebase][14], refactoring the code as I went, simplified the API, and made it extensible. You're now able to:

- Use it, since [I've open sourced it][15] (under MIT)
- Pick any email sending service, not just Mandrill, or bake your own
- Pick any template engine, or bake your own
- Get a default email service, just provide an API key
- Get a default template engine for free
- Get a default layout template out the box
- Customize it using styles that better represent your brand

Want to read some code?

```
var campaign = require('campaign');
var client = campaign({
    provider: campaign.providers.terminal()
});
var template = '<p>Your password reset key is: {{reset}}</p>';
var model = {
    to: 'someone@important.com',
    subject: 'Password Reset',
    reset: 'q12jFbwJsCKm'
};

client.sendString(template, model, done);

function done () {
  console.log('Done.');
}

```

The above will compile the template with Mustache, using the model we've provided, and "send" that email using the `terminal` transport, which doesn't really send emails, it just prints the JSON on your terminal.

![output.png][16]

I'm pretty happy with how the code extraction turned out, and I hope to be able to deliver more focused libraries like this one in the future. [Go read the documentation][6], I spent _a lot of time_ writing that thing!

  [1]: https://github.com/andris9/Nodemailer "andris9/Nodemailer on GitHub"
  [2]: https://postmarkapp.com/ "Postmark Transactional Email"
  [3]: http://mandrill.com/ "Mandrill Email API"
  [4]: https://github.com/niftylettuce/node-email-templates "niftylettuce/node-email-templates on GitHub"
  [5]: https://github.com/visionmedia/ejs "visionmedia/ejs on GitHub"
  [6]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [7]: http://i.imgur.com/BGyQlmp.png
  [8]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [9]: http://mandrill.com/ "Mandrill Email API"
  [10]: https://github.com/janl/mustache.js "janl/mustache.js on GitHub"
  [11]: https://github.com/mailchimp/Email-Blueprints "mailchimp/Email-Blueprints on GitHub"
  [12]: http://help.mandrill.com/entries/21678522-How-do-I-use-merge-tags-to-add-dynamic-content- "How do I use merge tags to add dynamic content?"
  [13]: http://i.imgur.com/PQXuMfQ.png "Example email sent using campaign"
  [14]: https://github.com/bevacqua/ponyfoo "bevacqua/ponyfoo on GitHub"
  [15]: https://github.com/bevacqua/campaign "bevacqua/campaign on GitHub"
  [16]: http://i.imgur.com/bt8IUS9.png

[ponyfoo modules campaign]
