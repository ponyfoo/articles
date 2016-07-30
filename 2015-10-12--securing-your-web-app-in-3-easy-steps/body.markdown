# Step 1: Let CloudFlare Manage Your DNS

A couple of months back I moved `ponyfoo.com` to the [CloudFlare][1] DNS. Originally, I had done this to leverage their CDN services, so that my static resources would load faster. Their onboarding process UX is unlike anything I had ever seen with other DNS providers and similar services, and I promptly found myself moving several of my sites onto CloudFlare.

[![Adding ponyfoo.com to CloudFlare][2]][3]

Once you've entered your domain name, they'll pull all domain records from your current DNS provider, and at that point you just have to point your servers to the CloudFlare name servers. The entire process shouldn't take longer than 10 minutes.

For free, CloudFlare gives you:

- The ability to use `ALIAS` records on the apex domain _-- like `example.com`, without a subdomain_
- Distribution on their network, saving bytes for you and saving time for your users
- A secure version of your site served over `https`
- All of the above without having to change anything in your application!

[![CloudFlare DNS record for ponyfoo.com][4]][5]

# Step 2: Fix Mixed Content Warnings

Like I said earlier, I've had `ponyfoo.com` traffic passing through CloudFlare for quite some time before deciding to turn it into a more secure site. To make the switch, a problem that frequently springs up is having _"mixed content warnings"_. This may happen when your page is served securely but other parts of the page aren't.

In my case, I had a number of issues that I had to resolve.

- I was loading fonts from `http://fonts.googleapis.com`, and those requests were getting blocked. I fixed that by switching over to the secure version
- I was loading gravatar icons over `http`, but again there's a secure version over `https`
- The vast majority of images in my articles are hosted on **imgur**. They provide secure versions of all their images but I was always pointing at the `http` versions, so I had to tweak some database entries in there
- There were a few other occurrences and you can find the entire changeset in a single commit on [`ponyfoo/ponyfoo`][6]

I always made it a point to link to articles relatively (e.g `/foo` instead of `http://pony.com/foo`), this was quite helpful because I didn't have to go around changing links everywhere. Similarly, I centralized the domain name in a single environment variable I use in places like emails and whatnot, so switching those over to `https` was a matter of adding a character to the configuration value.

The last place where I had to make a modification was comments, there weren't any comments with images from somewhere else than **imgur** _(most of those were uploaded by me, too)_, so I just changed the database again. The one thing I did have to add was a validation rule so that people can't submit comments with images served over `http`. There are more robust alternatives, such as detecting those links and _re-uploading them_ to imgur and then serving them securely, but alas, people hardly upload any images to this blog, so it wasn't worth the effort.

# Step 3: Enforce `https` on CloudFlare

After you've made sure that all your pages have a green lock and you don't have any more mixed content warnings, it's time to flip the switch. The easiest way to turn all your traffic into `https` without making any code modifications to your site or services at all is to go on to your CloudFlare account and add a redirection Page Rule that enforces `https`.

![Adding the Page Rule to turn your traffic green][7]

Done. You didn't have to change your site besides making sure that all external content loads securely, and now your site has that coveted green lock. Woohoo!

I'm taking steps into securing my site gradually, and if you're serious about security I would recommend that you look into the matters enumerated below as well. If you're a Node.js user, I suggest you look at [`helmet`][9] as a way to get started with all these.

- Enforcing TLS 1.2
- Enabling HTTP Strict Transport Security
- Enabling HTTP Public Key Pinning
- Enabling Content Security Policy
- Enabling X-XSS-Protection

When it comes to UX, now I can **finally** implement stuff like `ServiceWorker` _(for offline-first)_ -- which is only available over `https` in most browser implementations -- so keep an eye out for that!

Below is a screenshot of Pony Foo in all its green-padlocked glory. I'll probably wait until I can get a certificate of my own through the [Let's Encrypt][10] program, but they haven't launched yet. Soon, hopefully!

![Pony Foo in all its green padlocked glory][8]

  [1]: https://www.cloudflare.com/ "CloudFlare: 'Give us five minutes and we'll supercharge your website.'"
  [2]: https://i.imgur.com/uROcfFH.png
  [3]: https://www.cloudflare.com/ "CloudFlare: 'Give us five minutes and we'll supercharge your website.'"
  [4]: https://i.imgur.com/JrFackd.png
  [5]: https://www.cloudflare.com/ "CloudFlare: 'Give us five minutes and we'll supercharge your website.'"
  [6]: https://github.com/ponyfoo/ponyfoo/commit/cff0edc5c36efa224877a4ccb26c25eecee6327c "Commit cff0edc5c36efa224877a4ccb26c25eecee6327c on GitHub"
  [7]: https://i.imgur.com/n0YCo6H.png
  [8]: https://i.imgur.com/NSmVVzi.png
  [9]: https://github.com/helmetjs/helmet "helmetjs/helmet on GitHub"
  [10]: https://letsencrypt.org/ "Let’s Encrypt is a new Certificate Authority"
