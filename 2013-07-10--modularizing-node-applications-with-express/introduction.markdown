In this article, I want to talk about the big picture: **how to separate concerns of applications on different sub-domains in a clean manner**.

Sometimes, you need to deal with requests on two separate sub-domains in your application, say `www` and `blog`, but more often than not, this happens within the _same express application_. While it's _certainly possible_ to handle the routing using a single module, I've come to realize **it's best to use a modular approach**, similar to what [connect.vhost][1] does, but taken up a notch!

[1]: http://www.senchalabs.org/connect/vhost.html
