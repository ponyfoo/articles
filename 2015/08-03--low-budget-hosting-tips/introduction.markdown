I've been hosting Pony Foo on an AWS [`t1.micro`][1] instance for as long as I can remember. When it comes to AWS, _"low-budget"_ is **an oxymoron** -- as the cheapest type of instance on demand is priced somewhere near **$30/mo**. While the cheapest instance is pricey, it provides a far better experience than say, the lowest tiers of hosting on Heroku, which I've always found clumsy and quirky to use.

What I did like about Heroku back in the day I used it intensively was [their approach to app development][2] and how that translated into immutable infrastructure and deployments. I did learn tons and tons by following [12factor][2], and recently winded up writing up [my own immutable deployment system][3], using Packer, the AWS CLI, and plain old `bash`.

All things considered though, I've learned far more from using _IaaS-style_ hosting platforms like AWS than I did from Heroku. Using AWS forced me to learn more `bash` commands and UNIX\_-y\_ things than I ever had to worry about with Heroku. Learning how to come up with immutable deployments was also invaluable as it also gave me insight into how PaaS platforms work under the hood. Being able to tweak the instance exactly to your liking is invaluable too, and most PaaS really limit what you can do with their instances.

> Here, I'll share a few tips and tricks I've found useful when running applications using entirely custom scripts.

[1]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts_micro_instances.html
[2]: http://12factor.net/
[3]: /articles/leveraging-immutable-deployments
