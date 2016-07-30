<h1>Implementing OpenSearch</h1>

<p><kbd>seo</kbd> <kbd>opensearch</kbd></p>

<blockquote><p><a href="http://www.opensearch.org/" target="_blank">OpenSearch</a> is an specification that allows websites to improve <em>usability</em>. When implemented, it allows consumers to search your site <em>the way you intended them to</em>. All &#x2026;</p></blockquote>

<div><p><a href="http://www.opensearch.org/" target="_blank">OpenSearch</a> is an specification that allows websites to improve <em>usability</em>. When implemented, it allows consumers to search your site <em>the way you intended them to</em>. All major browsers support OpenSearch. <strong>Google Chrome</strong> for instance, allows users to search <em>OpenSearch-enabled</em> websites by using tab in the search bar.</p></div>

<div></div>

<div></div>

<div><h2 id="the-standard">The Standard</h2> <p>The first thing you need to do, is compose an XML file that conforms to the <strong>OpenSearch standard</strong>. I&#x2019;ll just provide an example of how I implemented it <em>in this blog</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-pi">&lt;?xml version=&quot;1.0&quot; encoding=&quot;UTF-8&quot;?&gt;</span>
<span class="md-code-tag">&lt;<span class="md-code-title">OpenSearchDescription</span> <span class="md-code-attribute">xmlns</span>=<span class="md-code-value">&quot;http://a9.com/-/spec/opensearch/1.1/&quot;</span> <span class="md-code-attribute">xmlns:moz</span>=<span class="md-code-value">&quot;http://www.mozilla.org/2006/browser/search/&quot;</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">ShortName</span>&gt;</span>Pony Foo<span class="md-code-tag">&lt;/<span class="md-code-title">ShortName</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">Description</span>&gt;</span>Search Pony Foo: Ramblings of a degenerate coder<span class="md-code-tag">&lt;/<span class="md-code-title">Description</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">InputEncoding</span>&gt;</span>UTF-8<span class="md-code-tag">&lt;/<span class="md-code-title">InputEncoding</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">Image</span> <span class="md-code-attribute">height</span>=<span class="md-code-value">&quot;16&quot;</span> <span class="md-code-attribute">width</span>=<span class="md-code-value">&quot;16&quot;</span> <span class="md-code-attribute">type</span>=<span class="md-code-value">&quot;image/x-icon&quot;</span>&gt;</span>http://blog.ponyfoo.com/favicon.ico<span class="md-code-tag">&lt;/<span class="md-code-title">Image</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">Url</span> <span class="md-code-attribute">type</span>=<span class="md-code-value">&quot;text/html&quot;</span> <span class="md-code-attribute">method</span>=<span class="md-code-value">&quot;get&quot;</span> <span class="md-code-attribute">template</span>=<span class="md-code-value">&quot;http://blog.ponyfoo.com/search/{searchTerms}&quot;</span> /&gt;</span>
<span class="md-code-tag">&lt;/<span class="md-code-title">OpenSearchDescription</span>&gt;</span>
</code></pre> <p>The <code class="md-code md-code-inline">{searchTerms}</code> placeholder will be replaced with the user&#x2019;s query.</p> <p>Next, all you have to do is reference your OpenSearch file somewhere, and reference it in site home, like this:</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-tag">&lt;<span class="md-code-title">link</span> <span class="md-code-attribute">rel</span>=<span class="md-code-value">&apos;search&apos;</span> <span class="md-code-attribute">type</span>=<span class="md-code-value">&apos;application/opensearchdescription+xml&apos;</span> <span class="md-code-attribute">title</span>=<span class="md-code-value">&apos;Pony Foo&apos;</span> <span class="md-code-attribute">href</span>=<span class="md-code-value">&apos;/opensearch.xml&apos;</span> /&gt;</span>
</code></pre> <p>You can see OpenSearch in action at <a href="http://imdb.com/" target="_blank" aria-label="IMDb">IMDb.com</a>, <a href="http://stackoverflow.com/" target="_blank" aria-label="Stack Overflow">StackOverflow</a>, or <strong>right here</strong>.</p></div>
