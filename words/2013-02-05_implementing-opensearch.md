# Implementing OpenSearch #

[OpenSearch](http://www.opensearch.org/ "Official Site") is an specification that allows websites to improve _usability_. When implemented, it allows consumers to search your site _the way you intended them to_. All major browsers support OpenSearch. **Google Chrome** for instance, allows users to search _OpenSearch-enabled_ websites by using tab in the search bar.

## The Standard ##

The first thing you need to do, is compose an XML file that conforms to the **OpenSearch standard**. I'll just provide an example of how I implemented it _in this blog_.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/" xmlns:moz="http://www.mozilla.org/2006/browser/search/">
    <ShortName>Pony Foo</ShortName>
    <Description>Search Pony Foo: Ramblings of a degenerate coder</Description>
    <InputEncoding>UTF-8</InputEncoding>
    <Image height="16" width="16" type="image/x-icon">http://blog.ponyfoo.com/favicon.ico</Image>
    <Url type="text/html" method="get" template="http://blog.ponyfoo.com/search/{searchTerms}" />
</OpenSearchDescription>
```

The `{searchTerms}` placeholder will be replaced with the user's query.

Next, all you have to do is reference your OpenSearch file somewhere, and reference it in site home, like this:

```html
<link rel='search' type='application/opensearchdescription+xml' title='Pony Foo' href='/opensearch.xml' />
```

You can see OpenSearch in action at [IMDb.com](http://imdb.com "IMDb"), [StackOverflow](http://stackoverflow.com "Stack Overflow"), or **right here**.
