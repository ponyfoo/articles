Getting started with the [Elastic Stack][v5] can feel a bit overwhelming. You need to set up `elasticsearch`, Kibana, and `logstash` before you even can get to the fun parts -- maybe even before you fully understand how the three synergize providing you with formidable and formerly untapped insights into your platform.

I have been running Pony Foo since late 2012, and, _although I had set up Google Analytics and Clicky since the very beginning_, these kinds of user tracking systems are far from ideal when it comes to troubleshooting. In the years since I launched the blog I tried a couple of instrumentation tools that would provide me with reporting and metrics from the Node.js application for my blog, but these solutions would require me to, well... *instrument* the Node apps by patching them with a snippet of code that would then communicate with a third party service. Visiting that service I could learn more about the current state of my production application. For a variety of reasons, _-- too expensive, makes my app too slow, etc. --_ I ended up ditching every one of these solutions not long after giving them a shot.

Not long ago I wrote about [setting up search for a blog][setup], and since I'm already working on the Kibana analytics dashboard team I figured it wouldn't hurt to learn more about `logstash` so that I could go full circle and finally have a look at some server metrics my way.

**Logstash is, in short, a way to get data from one place to another.** It helps stream events pulled out of files, HTTP requests, tweets, event logs, or [dozens of other input sources][inputs]. After processing events, Logstash can output them via Elasticsearch, disk files, email, HTTP requests, or [dozens of other output plugins][outputs].

[![Logstash inputs and outputs][inout]][intro]

The above graph shows how Logstash can provide tremendous value through a relatively simple interface where we define inputs and outputs. You could easily use it to pull a Twitter firehose of political keywords about a presidential campaign into Elasticsearch for further analysis. Or to anticipate breaking news on Twitter as they occur.

Or maybe you have more worldly concerns, like streaming log events from `nginx` and `node` into `elasticsearch` for increased visibility through a Kibana dashboard. That's what we're going to do in this article.

[v5]: https://www.elastic.co/v5 "Say “Heya” to the Elastic Stack"
[setup]: /articles/setting-up-elasticsearch-for-a-blog "Setting Up Elasticsearch for a Blog on Pony Foo"
[inputs]: https://www.elastic.co/guide/en/logstash/current/input-plugins.html "Logstash input plugins"
[outputs]: https://www.elastic.co/guide/en/logstash/current/output-plugins.html "Logstash output plugins"
[inout]: https://i.imgur.com/2nevyPE.png "logstash.png"
[intro]: https://www.elastic.co/guide/en/logstash/current/introduction.html "Introduction to Logstash"
