This detailed article series aims to explain:

* How to provision a ready-made image _before every deployment_
* How to make that image set up `nginx`, `node`, and your web application
* How to dynamically update DNS record sets
* How to let AWS handle scaling on your behalf
* How to **avoid downtime** during deployments
* How to clean up all this mess
* How to do all of the above in **plain old bash**
* _Why any of the above matters_

In this article **I'll start by explaining why doing any of this matters**, and then move on to _creating [immutable][1] images_ ready for deployment using [Packer][2].

[1]: http://chadfowler.com/blog/2013/06/23/immutable-deployments/
[2]: http://packer.io/
