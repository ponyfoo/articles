I just finished reading [The Lean Startup](http://www.amazon.com/dp/0307887898 "The Lean Startup book, by Eric Ries"), an immersive, ground-breaking take on entrepreneurship and startup management, with views heavily rooted in the Toyota Production System. There are quite a few take-aways for us, the **web work-force**, to learn.

The Lean Startup introduces us to a series of practices we can follow in order to build products faster, get feedback earlier and more often, and most importantly, **develop features customers actually want**. In this article, I'll go over the fundamentals, focusing on how Lean Development affects software development, not just startup management.

[![lean-startup.jpg][1]](http://www.amazon.com/dp/0307887898 "The Lean Startup book, by Eric Ries")

Here is an [infographic with the key take-aways](http://thumbnails.visually.netdna-cdn.com/the-lean-startup_50291668aa9bb.png "Click to view") from the Lean Startup model.

I've put together a recap of the practices that'll help transform your development process into a more _sincerely results-oriented methodology_. These are, in my humble opinion, the most important factors of a Lean Startup, when it comes to software builders.

# Prototyping an MVP

Early in, we might want to approach things differently. If we have a product idea, we don't want to be building a fully-featured product that might fail the moment we start publicizing it. Instead, what we should do, is put together a **Minimum Viable Product** (MVP for short), that will help us figure whether we are going in the right direction before we get to actually building anything.

An [MVP](http://en.wikipedia.org/wiki/Minimum_viable_product "Minimum Viable Product") might be as simple as a smoke test, or it might contain just the essential features for us to give the concept a test drive and gather data.

> The canonical MVP strategy for a web application is to create a mock website for the product and purchase online advertising to direct traffic to the site. The mock website may consist of a marketing landing page with a link for more information or purchase. The link is not connected to a purchasing system, instead clicks are recorded and measure customer interest.

# Split Testing

Every new feature that's to be introduced into the produce should be pitted against the existing product. We should be gathering metrics about each of the different scenarios. Much like running an experiment, we test to see if the users are more engaged when the new feature is available to them.

Split testing (also known as [A/B testing](http://en.wikipedia.org/wiki/A/B_testing "A/B Product Testing")) consists of providing a small portion of your customer-base with a different version of the product, where the only change from the current version of the product is the new feature. The metrics and analytics we pick up from running an experiment should give us enough feedback to make an informed decision about whether we will be _including the feature_ in our final product, or **discarding it altogether**.

This approach leads to something called _validated learning_, and that means we make _well-informed_ decisions about whether we'll include any feature, and we won't be just going by our gut feel or by what a dictator in our company imposes.

Here is a lengthy, thorough article on the [do's and don'ts of A/B testing](http://www.smashingmagazine.com/2010/06/24/the-ultimate-guide-to-a-b-testing/ ""), featured on Smashing Magazine.

# Not Just Split Testing

Think of split testing as forking. Now that you've learned that features should go out on a one-by-one basis, and be tested independently, it makes sense as developers, to keep these features also strictly separated in our repositories. To that end, we should be branching off of the main branch into **feature branches**, and using one branch per feature. In this way, we can also shorten our cycles by making every feature way easier to plug in and out of our main product.

Once our feature is approved or rejected, we can merge it into the main branch, delete it, or keep working on it, maybe attempting a different take on the same feature.

# Cross-functional Teams

Using a validated learning approach based on split testing, it is best to work as cross-functional teams.

This means we should be working with small teams. Ideally, we should be able to compose and decompose these teams as needed when developing new features. Cross-functional teams should include all the required people in order to design, develop, and deploy a feature, complete and with a split test.

# Continuous Deployment

Lean development allows us to iterate more quickly on our process, having a continuous deployment process in place will allow us to gather feedback more quickly.

Gathering feedback early is of the _utmost importance_ because **we don't want to be building a product nobody wants or a feature nobody intends to use**.

Continuous deployment further enables us to push bug fixes and updates far quicker than what we are used to in more traditional development processes, and with _far less friction_, too.

# Error Correction

The book suggests setting the project up with someone in charge of root cause analysis when things go wrong. This goes a great length to help us get the team involved in delivering a high quality product, and together with the suggestion to rotate the person in charge, can go great lengths in bringing teams closer together, working as a unit.

But don't take my word for it, **read the book instead**.

P.S Kanban, [on the other hand](http://blog.aha.io/index.php/kanban-the-secret-engineer-killer/ "Kanban - The Secret Engineer Killer"), should be taken with a pinch of salt.

  [1]: http://i.imgur.com/9wRlDts.jpg
