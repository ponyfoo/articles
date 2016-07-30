Given the [immutable images we're now able to build][1], thanks to the last article, in this article we'll lay out an architecture, based on **Amazon Web Services**, that's able to take advantage of those immutable images.

# Architecture Overview

Our journey starts at [Route 53][2], a DNS provider service from Amazon. You'll set up the initial DNS configuration by hand. I'll leave the `NS`, `SOA`, and `TXT` records up to you. The feature that interests me in Route 53 is [creating `ALIAS` record sets][3]. These special record sets allow you to bind a domain name to an [Elastic Load Balancer _(ELB)_][4], which can then distribute traffic among your [Elastic Cloud Compute _(EC2)_][5] server instances.

The eye candy in `ALIAS` record sets is that you're able to use them at the naked, or apex domain level _(e.g `ponyfoo.com`)_, and not just sub-domains _(e.g `blog.ponyfoo.com`)_. Our scripts will set up these `ALIAS` records on our behalf whenever we want to deploy to a new environment, also a very enticing feature.

> When you finish reading this article, you'll be able to run the commands below and have your application listening at `dev01.example.com` when they complete, without the need for any manual actions.
>     
>     NODE_ENV=dev01 npm run setup
>     NODE_ENV=dev01 npm run deploy
>     
> 
> 

We don't merely use [ELB][4] because of its ability to route traffic from naked domains, but also because it enables us to have many web servers behind a single domain, which makes our application more robust and better able to handle load, becoming _highly available_.

In order to better manage the instances behind our [ELB][4], we'll use [Auto Scaling Groups _(ASG)_][6]. These come at no extra cost, you just pay for the [EC2][5] instances in them. The [ASG][6] will automatically detect _"unhealthy"_ [EC2][5] instances, or instances that the [ELB][4] has deemed unreachable after pinging them with `GET` requests. Unhealthy instances are automatically disconnected from the [ELB][4], meaning they'll no longer receive traffic. However, we'll enable [connection draining][7] so that they gracefully respond to pending requests before shutting down.

When a new deployment is requested, we'll create a new [ASG][6], and provision it with the new `carnivore` AMI, [which you may recall from our last encounter][1]. The new [ASG][6] will spin as many [EC2][5] instances as desired. We'll wait until [EC2][5] reports every one of those instances are properly initialized and healthy. We'll then wait until [ELB][4] reports every one of those instances is reachable via `HTTP`. This ensures that no downtime occurs during our deployment. When every new instance is reachable on [ELB][4], we'll remove the outdated [EC2][5] instances from [ELB][4] first, and _downscale the outdated ASG to 0_. This will cause the ASG to allow connection draining to kick in on the outdated instances, and terminate them afterwards. Once all of that is over, we delete the outdated ASG.

> This approach might not be blazing fast, but I'll take a speed bump over downtime any day.

Of course, none of this would be feasible if spinning a new instance took 15 minutes while installing software that should've been baked into an image. This is why creating those images was crucial. In a slow process such as this, baking images saves us much appreciated startup time.

[1]: /articles/immutable-deployments-packer
[2]: http://aws.amazon.com/route53/
[3]: http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
[4]: http://aws.amazon.com/elasticloadbalancing/
[5]: http://aws.amazon.com/ec2/
[6]: http://aws.amazon.com/autoscaling/
[7]: https://aws.amazon.com/blogs/aws/elb-connection-draining-remove-instances-from-service-with-care/
