This article describes in detail the steps I took in setting up Elasticsearch as the search provider for Pony Foo. I start by explaining what Elasticsearch is, how you can set it up to make useful searches through the Node.js API client, and how to deploy the solution onto a Debian or Ubuntu environment.

A while back I started working at Elastic -- the [company behind Elasticsearch][es], a search engine & realtime analytics service powered by Lucene indexes. It's an **extremely exciting open-source company** and I'm super happy here _-- and we're hiring, drop me a note!_ 😉

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">🎉 Thrilled to announce I&#39;ve started working at <a href="https://twitter.com/elastic">@elastic</a>!<br>⛹ Working on Kibana (ES graphs)<br>👌 Great fun/team! Hiring!<br>🎢 <a href="https://t.co/8u8B4o5IFf">https://t.co/8u8B4o5IFf</a></p>&mdash; Nicolás Bevacqua (@nzgb) <a href="https://twitter.com/nzgb/status/714803063958077441">March 29, 2016</a></blockquote>

Possible use cases for Elasticsearch range from indexing millions of HTTP log entries, analyzing public traffic incidents in real-time, streaming tweets, all the way to tracking and predicting earthquakes and back to providing search for a lowly blog like Pony Foo.

We also build [Kibana][kibana], a dashboard that sits in front of Elasticsearch and lets you perform and graph the most complex queries you can possibly imagine. Many use Kibana across those cool service status flat screens in hip offices across San Francisco.

[![Image of Kibana on flat screens across an office.][kbn]][kibana]

But enough about me and the cool things you can do with Elastic's products. Let's start by talking about Elasticsearch in more meaningful, technical terms.

[es]: https://www.elastic.co/products/elasticsearch "Elasticsearch product overview"
[kbn]: https://i.imgur.com/6lNsZrT.jpg
[kibana]: https://github.com/elastic/kibana "elastic/kibana on GitHub"
