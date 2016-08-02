# What is Elasticsearch, even? 🔎

Elasticsearch is a REST HTTP service that wraps around [Apache Lucene][luc], a Java-based indexing and search technology that also features spellchecking, hit highlighting and advanced analysis/tokenization capabilities. On top of what Lucene already provides, Elasticsearch adds an HTTP interface, meaning you don't need to build your application using Java anymore; and is distributed by default, meaning you won't have any trouble scaling your operations to thousands of queries per second.

Elasticsearch is great for setting up blog search because you could basically dump all your content into an index and have them deal with user's queries, with very little effort or configuration.

Here's how I did it.

# Initial Setup

I'm on a Mac, so _-- for development purposes --_ I just installed `elasticsearch` using [Homebrew][brw].

```shell
brew install elasticsearch
```

If you're not on a Mac, just go to the download page and [get the latest version][dl], unzip it, run it in a shell, and you're good to go.

[![Installation steps for Elasticsearch][steps]][dl]

Once you have the `elasticsearch` executable, you can run it on your terminal. Make sure to leave the process running while you're working with it.

```shell
elasticsearch
```

Querying the index is a matter of using `curl`, which is a great diagnostics tool to have a handle on; a web browser, by querying `http://localhost:9200` _(`9200` is the port Elasticsearch listens at by default)_; the [Sense][sns] Chrome extension, which provides a simple interface into the Elasticsearch REST service, or the [Console plugin for Kibana][kibana-console], which is similar to Sense.

There are client libraries that consume the HTTP REST API available to several different languages. In our case, we'll use the Node.js client: [`elasticsearch`][npm-es].

```shell
npm install --save elasticsearch
```

The `elasticsearch` API client is quite pleasant to work with, they provide both `Promise`-based and callback-based API through the same methods. First off, we'll create a client. This will be used to talk to the REST service for our Elasticsearch instance.

# Creating an Elasticsearch Index

We'll start by importing the `elasticsearch` package and instantiating a REST client configured to print all logging statements.

```js
import elasticsearch from 'elasticsearch';
const client = new elasticsearch.Client({
  host: 'http://localhost:9200',
  log: 'debug'
});
```

Now that we have a `client` we can start interacting with our Elasticsearch instance. We'll need an index where we can store our data. You can think of [an Elasticsearch index][what] as the rough equivalent of a database instance. A huge difference, though, is that **you can very easily query multiple Elasticsearch indices** at once _-- something that's not trivial with other database systems._

I'll create an index named `'ponyfoo'`. Since `client.indices.create` returns a `Promise`, we can `await` on it for our code to stay easy to follow. If you need to brush up on `async` / `await` you may want to read ["Understanding JavaScript’s async await"][asaw] and the [article on Promises][prom] as well.

```js
await client.indices.create({ index: 'ponyfoo' });
```

That's all the setup that is **required**.

# Creating an Elasticsearch Mapping

In addition to creating an index, you can *optionally* create [an explicit type mapping][mapdoc]. Type mappings aid Elasticsearch's querying capabilities for your documents -- avoiding issues when you are storing dates using their timestamps, [among other things][map].

If you don't create an explicit mapping for a type, Elasticsearch will **infer field types based on inserted documents** and create a dynamic mapping.

> _A timestamp is often represented in JSON as a `long`, but Elasticsearch will be unable to detect the field as a `date` field, preventing date filters and facets such as the date histogram facet from working properly._  
> _--- [Elasticsearch Documentation][mapdoc]_

Let's create a mapping for the type `'article'`, which is the document type we'll use when storing blog articles in our Elasticsearch index. Note how even though the `tags` property will be stored as an array, Elasticsearch takes care of that internally and we only need to specify that each tag is of type string. The `created` property will be a `date`, as hinted by the mapping, and everything else is stored as strings.

```js
await client.indices.putMapping({
  index: 'ponyfoo',
  type: 'article',
  body: {
    properties: {
      created: { type: 'date' },
      title: { type: 'string' },
      slug: { type: 'string' },
      teaser: { type: 'string' },
      introduction: { type: 'string' },
      body: { type: 'string' },
      tags: { type: 'string' }
    }
  }
});
```

The remainder of our initial setup involves two steps -- both of them involving keeping the Elasticsearch index up to date, so that querying it yields meaningful results.

- Importing all of the current articles into our Elasticsearch index
- Updating the Elasticsearch index whenever an article is updated or a new article is created

# Keeping Elasticsearch Up-to-date
These steps vary slightly depending on the storage engine you're using for blog articles. For Pony Foo, I'm using MongoDB and the `mongoose` driver. The following piece of code will trigger a post-save hook whenever an article is saved _-- regardless of whether we're dealing with an insert or an update._

```js
mongoose.model('Article').schema.post('save', <mark>updateIndex</mark>);
```

The `updateIndex` method is largely independent of the storage engine: our goal is to update the Elasticsearch index with the updated document. We'll be using the `client.update` method for an article of `id` equal to the `_id` we had in our MongoDB database, although that's entirely up to you -- I chose to reuse the MongoDB, as I found it most convenient. The provided `doc` should match the type mapping we created earlier, and as you can see I'm just forwarding part of my MongoDB document to the Elasticsearch index.

Given that we are using the `doc_as_upsert` flag, a new document will be inserted if no document with the provided `id` exists, and otherwise the existing `id` document will be modified with the updated fields, again in a single HTTP request to the index. I could've done `doc: article`, but I prefer **a whitelist approach** where I explicitly name the fields that I want to copy over to the Elasticsearch index, which explains the `toIndex` function.

```js
const id = article._id.toString();
await <mark>client.update</mark>({
  index: 'ponyfoo',
  type: 'article',
  id,
  body: {
    <mark>doc</mark>: toIndex(article),
    <mark>doc_as_upsert: true</mark>
  }
});
function toIndex (article) {
  return {
    created: article.created,
    title: article.title,
    slug: article.slug,
    teaser: article.teaser,
    introduction: article.introduction,
    body: article.body,
    tags: article.tags
  };
}
```

Whenever an article gets updated in our MongoDB database, the changes will be mirrored onto Elasticsearch. That's great for new articles or changes to existing articles, but what about articles that existed before I started using Elasticsearch? Those wouldn't be in the index unless I changed each of them and the post-save hook picks up the changes and forwards them to Elasticsearch.

# Wonders of the Bulk API, or Bootstrapping an Elasticsearch Index

To bring your Elasticsearch index up to date with your blog articles, you will want to use the [bulk operations API][bulk], which allows you to perform several operations against the Elasticsearch index in one fell swoop. The bulk API consumes operations from an array under the `[cmd_1, data_1?, <mark>cmd_2, data_2?</mark>, ..., cmd_n, data_n?]` format. The question marks note that the data component of operations is optional. Such is the case of `delete` commands, which don't require any additional data beyond an object `id`.

Provided an array of `articles` pulled from MongoDB or elsewhere, the following piece of code reduces `articles` into command/data pairs on a single array, and submits all of that to Elasticsearch as a single HTTP request through its bulk API.

```js
await client.bulk({
  body: articles.reduce(toBulk, [])
});

function toBulk (body, article) {
  body.push({
    <mark>update</mark>: {
      _index: 'ponyfoo',
      _type: 'article',
      _id: article._id.toString()
    }
  });
  body.push({
    <mark>doc</mark>: toIndex(article),
    <mark>doc_as_upsert: true</mark>
  }); // <mark>toIndex</mark> from previous code block
  return body;
}
```

If JavaScript had [`.flatMap`][flatmap] we could do away with `.reduce` and `.push`, but we're not quite there yet.


```js
await client.bulk({
  body: articles<mark>.flatMap(</mark>article => [{
    update: {
      _index: 'ponyfoo',
      _type: 'article',
      _id: article._id.toString()
    }
  }, {
    doc: toIndex(article),
    doc_as_upsert: true
  }])
});
```

Great stuff! 🎉

Up to this point we have:

- Installed Elasticsearch and the `elasticsearch` npm package
- Created an Elasticsearch index for our blog
- Created an Elasticsearch mapping for articles
- Set up a hook that upserts articles when they're inserted or updated in our source store
- Used the bulk API to pull all articles that weren't synchronized into Elasticsearch yet

We're still missing the awesome parts ✨, though! 

- Set up a `query` function that takes some options and returns the articles matching the user's query
- Set up a `related` function that takes an `article` and returns similar articles
- Create an automated deployment script for Elasticsearch

*Shall we?*

# Querying the Elasticsearch Index

While utilizing the results of querying the Elasticsearch index is out of the scope of this article, you probably still want to know how to write a function that can query the engine you so carefully set up with your blog's amazing contents.

A simple `query(options)` function looks like below. It returns a `Promise` and it uses `async` / `await`. The resulting search hits are mapped through a function that only exposes the fields we want. Again, we take a whitelisting approach as favored earlier when we inserted documents into the index. Elasticsearch offers [a querying DSL][dsl] you can leverage to build complex queries. For now, we'll only use the [`match` query][mat] to find articles whose `title` match the provided `options.input`.

```js
async function query (options) {
  const result = await client.search({
    index: 'ponyfoo',
    type: 'article',
    body: {
      query: {
        <mark>match</mark>: {
          <mark>title</mark>: options.input
        }
      }
    }
  });
  return <mark>result.hits.hits.map(searchHitToResult)</mark>;
}
```

The `searchHitToResult` function receives the raw search hits from the REST Elasticsearch API and maps them to simple objects that contain only the `_id`, `title`, and `slug` fields. In addition, we'll include the `_score` field, Elasticsearch's way of telling us *how confident we should be* that the search hit reliably matches the human's query. Typically more than enough for dealing with search results.

```js
function searchHitToResult (hit) {
  return {
    _score: hit._score,
    _id: hit._id,
    title: hit._source.title,
    slug: hit._source.slug
  };
}
```

> You could always query the MongoDB database for `_id` to pull in more data, such as the contents of an article.

Even in the case of a simple blog, you wouldn't consider a search solution sufficient if users could only find articles by matching their titles. You'd want to be able to filter by tags, and even though the article titles should be valued higher than their contents _(due to their prominence)_, you'd still want users to be able to search articles by querying their contents directly. You probably also want to be able to specify date ranges, and then expect to see results only within the provided date range.

What's more, you'd expect to be able to fit all of this in a single querying function.

# Building Complex Elasticsearch Queries

As it turns out, we don't have to drastically modify our `query` function to this end. Thanks to the rich querying DSL, our problem becomes finding out which types of queries we need to use, and figuring out how to stack the different parts of our query.

To begin, we'll add the ability to query several fields, and not just the `title`. To do that, we'll use the [`multi_match` query][mulmat], adding `'teaser', 'introduction', 'content'` to the `title` we were already querying about.

```js
async function query (options) {
  const result = await client.search({
    index: 'ponyfoo',
    type: 'article',
    body: {
      query: {
        <mark>multi_match</mark>: {
          query: options.input,
          fields: <mark>['title', 'teaser', 'introduction', 'content']</mark>
        }
      }
    }
  });
  return result.hits.hits.map(searchHitToResult);
}
```

Earlier, I brought up the fact that I want to rate the `title` field higher. In the context of search, this is usually referred to as giving a term more "weight". To do this through the Elasticsearch DSL, we can use the `^` field modifier to boost the `title` field three times.

```
{
  query: {
    multi_match: {
      query: options.input,
      fields: [<mark>'title^3'</mark>, 'teaser', 'introduction', 'content']
    }
  }
}
```

If we have additional filters to constrain a query, I've found that the most effective way to express that is using [a `bool` query][bool], moving the `filter` options into a function and placing our existing `multi_match` query under a `must` clause, within our `bool` query. Bool queries are a powerful querying DSL that allow for a recursive yet declarative and simple interface to defining complex queries.

```js
{
  query: {
    <mark>bool</mark>: {
      <mark>filter: filters(options)</mark>,
      <mark>must</mark>: {
        multi_match: {
          query: options.input,
          fields: ['title^3', 'teaser', 'introduction', 'content']
        }
      }
    }
  }
}
```

In the simplest case, the applied `filter` does nothing at all, leaving the original query unmodified. Here we return an empty `filter` object.

```js
function filters (options) {
  return <mark>{}</mark>;
}
```

When the user-provided `options` object contains a `since` date, we can use that to define a `range` for our filter. For the [`range` filter][rng] we can specify fields and a condition. In this case we specify that the `created` field must be `gte` _(**g**reater **t**han or **e**qual)_ the provided `since` date. Since we moved this logic to a `filters` function, we don't clutter the original `query` function with our _(albeit simple)_ filter-building algorithm. We place our filters in a `must` clause within a `bool` query, so that we can filter on as many concerns as we have to.

```js
function filters (options) {
  const clauses = [];
  if (options.since) {
    clauses.unshift(<mark>since(options.since)</mark>);
  }
  return all(clauses);
}
function all (clauses) {
  return <mark>{ bool: { must: clauses } }</mark>;
}
function since (date) {
  return <mark>{ range: { created: { gte: date } } }</mark>;
}
```

When it comes to constraining a query to a set of user-provided tags, we can add [a `bool` filter][bool] once again. Using the `must` clause, we can provided an array of `term` queries for the `tags` field, so that articles without one of the provided tags are filtered out. That's because we're specifying that the **query must match** each user-provided `tag` against the `tags` field in the article.

```js
function filters (options) {
  const tags = Array.isArray(options.tags) ? options.tags : [];
  const clauses = <mark>tags.map(tagToFilter)</mark>;
  if (options.since) {
    clauses.unshift(<mark>since(options.since)</mark>);
  }
  return all(clauses);
}
function all (clauses) {
  return { bool: { must: clauses } };
}
function since (date) {
  return { range: { created: { gte: date } } };
}
function tagToFilter (tag) {
  return { term: { tags: tag } };
}
```

We could keep on piling condition clauses on top of our `query` function, but the bottom line is that we can easily construct a query using the [Elasticsearch querying DSL][dsl], and it's most likely going to be able to perform the query we want within a single request to the index.

# Finding Similar Documents

The API to find related documents is quite simple as well. Using the [`more_like_this` query][mlt], we could specify the `like` parameter to look for articles related to a user-provided document _-- by default, a full text search is performed_. We could reuse the `filters` function we just built, for extra customization. You could also specify that you want at most `6` articles in the response, by using the `size` property.

```js
{
  query: {
    bool: {
      filter: filters(options),
      must: {
        <mark>more_like_this</mark>: {
          <mark>like</mark>: {
            _id: options.article._id.toString() 
          }
        }
      }
    }
  },
  <mark>size</mark>: 6
}
```

Using the `more_like_this` query we can quickly set up those coveted "related articles" that spring up on some blogging engines but feel so very hard to get working properly in your homebrew blogging enterprise.

The best part is that Elasticsearch took care of all the details for you. I've barely had to explain any search concepts at all in this blog post, and you came out with a powerful `query` function that's easily augmented, as well as the `body` of a search query for related articles _-- nothing too shabby!_

To round things out, I'll detail the steps I took in making sure that my deployments went smoothly with my recently added Elasticsearch toys.

# Rigging for Deployment 🚀

After figuring out the indexing and querying parts _(even though I now work at Elastic I'm pretty far from becoming a search demigod)_, and setting up the existing parts of the blog so that search and related articles leverage the new Elasticsearch services I wrote for [`ponyfoo/ponyfoo`][q], came deploying to production.

It took a bit of research to get the deployment right for Pony Foo's [_Debian Jessie_][jess] production environment. Interestingly, my biggest issue was figuring out how to install Java 8. The following chunk of code installs Java 8 in Debian Jessie and sets it as the default `java` runtime. Note that we'll need the cookie in `wget` so that Oracle validates the download.

```shell
echo "install java"
JAVA_PACK=jdk-8u92-linux-x64.tar.gz
JAVA_VERSION=jdk1.8.0_92
wget -nv --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u92-b14/$JAVA_PACK
sudo mkdir /opt/jdk
sudo tar -zxf $JAVA_PACK -C /opt/jdk
sudo update-alternatives --install /usr/bin/java java /opt/jdk/$JAVA_VERSION/bin/java 100
sudo update-alternatives --install /usr/bin/javac javac /opt/jdk/$JAVA_VERSION/bin/javac 100
```

> Before coming to this piece of code, I tried using `apt-get` but nothing I did seemed to work. The `oracle-java8-installer` package some suggest you should install was nowhere to be found, and the `default-jre` package isn't all that well supported by `elasticsearch`.

After installing Java 8, we have to install Elasticsearch. This step involved copying and pasting Elastic's installation instructions, for the most part.

```shell
echo "install elasticsearch"
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
sudo apt-get update
sudo apt-get -y install elasticsearch
```

Next up came setting up `elasticsearch` as a service that also relaunches itself across reboots.

```shell
echo "elasticsearch as a service"
sudo update-rc.d elasticsearch defaults 95 10
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

> I deploy **Pony Foo** through a series of [immutable deployments][imul], _(that article had [two parts][lev]! 🍀)_ building disk images along the way using Packer. For the most part, unless I'm setting up something like Elasticsearch, the deployment consists of installing the latest `npm` dependencies and updating the server to the latest version of the Node.js code base. More fundamental changes take longer, however, when I need to re-install parts of the system dependencies for example, but that doesn't occur as often. This leaves me with a decently automated deployment process while retaining tight control over the server infrastructure to use `cron` and friends as I see fit.

When I'm ready to fire up the `elasticsearch` service, I just run the following. The last command prints useful diagnostic information that comes in handy while debugging your setup.

```shell
echo "firing up elasticsearch"
sudo service elasticsearch restart || sudo service elasticsearch start || (sudo cat /var/log/elasticsearch/error.log && exit 1)
sudo service elasticsearch status
```

That's about it.

> If the whole deployment process feels too daunting for you, Elastic offers [Elastic Cloud][clo]. Although, at $45/mo, it's mostly aimed at companies! If you're flying solo, you might just have to strap on your keyboards and start fiercely smashing those hot keys.

There is one more step in my setup, which is that I hooked my application server up in such a way that the first search request creates the Elasticsearch index, type mapping, and bulk-inserts documents into the index. This could alternatively be done before the Node.js application starts listening for requests, but since it's not a crucial component of Pony Foo, that'll do for now!

# Conclusions

I had a ton of fun setting up Elasticsearch for the blog. Even though I already had a homebrew search solution, it performed very poorly and the results weren't anywhere close to accurate. With Elasticsearch the search results are much more on point, and hopefully will be more useful to my readers. Similarly, related articles should be more relevant now as well!

I can't wait to hook Elasticsearch up with [Logstash][ls] and start feeding `nginx` logs into my ES instance so that I can see some realtime HTTP request data _-- besides what Google Analytics has been telling me --_ **for the first time** since I started blogging back in late 2012. I might do this next, when I have some free time. Afterwards, I might set up some sort of public [Kibana][kibpro] dashboard displaying realtime metrics for Pony Foo servers. That should be fun!

[luc]: https://lucene.apache.org/ "“Apache Lucene and Solr set the standard for search and indexing performance”"
[brw]: http://brew.sh/ "“The missing package manager for OS X”"
[steps]: https://i.imgur.com/5ndK5vx.png
[dl]: https://www.elastic.co/downloads/elasticsearch "Download Elasticsearch"
[sns]: https://chrome.google.com/webstore/detail/sense-beta/lhjgkmllcaadmopgmanpapmpjgmfcfig?hl=en "Sense on the Chrome Web Extension"
[kibana-console]: https://www.elastic.co/guide/en/kibana/5.0/console-kibana.html "The Console plugin provides a UI to interact with the REST API of Elasticsearch"
[npm-es]: https://www.npmjs.com/package/elasticsearch "elasticsearch on npm"
[what]: https://www.elastic.co/blog/what-is-an-elasticsearch-index "What is an Elasticsearch Index?"
[map]: https://www.elastic.co/blog/found-elasticsearch-mapping-introduction "An Introduction to Elasticsearch Mapping"
[mapdoc]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/mapping.html "Mapping is the process of defining how a document, and the fields it contains, are stored and indexed"
[asaw]: /articles/understanding-javascript-async-await "Understanding JavaScript’s async await on Pony Foo"
[prom]: /articles/es6-promises-in-depth "ES6 Promises in Depth on Pony Foo"
[bulk]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/docs-bulk.html "“The bulk API makes it possible to perform many index/delete operations in a single API call. This can greatly increase the indexing speed.”"
[flatmap]: /articles/proposal-draft-for-flatten-and-flatmap "Proposal Draft for .flatten and .flatMap on Pony Foo"
[dsl]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl.html "“Elasticsearch provides a full Query DSL based on JSON to define queries. Think of the Query DSL as an AST of queries”"
[mat]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-match-query.html "“A family of match queries that accepts text/numerics/dates, analyzes them, and constructs a query.”"
[mulmat]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-multi-match-query.html "The multi_match query builds on the match query to allow multi-field queries"
[boo]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-multi-match-query.html#_literal_fields_literal_and_per_field_boosting "Per-field boosting"
[rng]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-range-query.html "Matches documents with fields that have terms within a certain range."
[bool]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-bool-query.html "A query that matches documents matching boolean combinations of other queries."
[mlt]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/query-dsl-mlt-query.html "“The More Like This Query (MLT Query) finds documents that are similar to a given set of documents.”"
[jess]: https://www.debian.org/releases/jessie/ "Debian Jessie Release"
[q]: https://github.com/ponyfoo/ponyfoo/blob/c7b6f069de4c0be7ee1a899218fc677ddf4f1d7d/services/articleElasticsearch.js#L27-L43 "ponyfoo/ponyfoo on GitHub"
[imul]: /articles/immutable-deployments-packer "Immutable Deployments and Packer on Pony Foo"
[lev]: /articles/leveraging-immutable-deployments "Leveraging Immutable Deployments on Pony Foo"
[ls]: https://www.elastic.co/products/logstash "Logstash Product Overview"
[kibpro]: https://www.elastic.co/products/kibana "Kibana Product Overview"
[clo]: https://www.elastic.co/cloud "Elastic Cloud Product Overview"
