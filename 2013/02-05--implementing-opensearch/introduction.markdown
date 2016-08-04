# The Standard

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
