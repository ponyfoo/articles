> This article was originally posted on the [Elastic company blog][elastic].

User interfaces for cloud services are typically boring, long lists of nodes, clusters, regions, proxies, users, snapshots, logs, and lists. Identifying patterns in the data is mostly a responsibility of the UI implementor, and not so much the consumer. When such an identifiable pattern emerges, implementors should come up with ways to break away from these long tabular data lists and display data in more valuable ways.

Tabular data is by its very nature repetitive and boring. There is nothing wrong with displaying boring data, but visualizations can certainly help identify outliers when we have long data lists.

The following screenshot shows a small fraction of a list of allocators, along with their health and available capacity. An allocator is a container that may host several Elasticsearch or Kibana Docker instances, and perhaps some available capacity. These allocators are sorted by available capacity, which becomes immediately obvious when we use a visualization to display how capacity is distributed in each allocator. The visualizations are relative to each other in width, so it's easier to understand the relationship between different allocators in terms of total capacity.

![A list of allocators and their respective node distributions.][1]

Note that this is just the end-result of what we arrived at after around two weeks of on-and-off iteration on the design and UX of the visualization. Let's go over that iteration process.

[1]: https://i.imgur.com/ZpyYMV1.png
[elastic]: https://www.elastic.co/blog/color-coded-visualizations-react
