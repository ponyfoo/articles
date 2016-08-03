<div></div>

<h1>ES6 JavaScript Destructuring in Depth</h1>

<p><kbd>es6</kbd> <kbd>destructuring</kbd> <kbd>es6-in-depth</kbd></p>

<blockquote><p>I&#x2019;ve briefly mentioned a few ES6 features <em>(and how to <a href="https://ponyfoo.com/articles/universal-react-babel#setting-up-babel">get started with Babel</a>)</em> in the React article series I&#x2019;ve been writing about, and now I want to <strong>focus &#x2026;</strong></p></blockquote>

<div><p>I&#x2019;ve briefly mentioned a few ES6 features <em>(and how to <a href="https://ponyfoo.com/articles/universal-react-babel#setting-up-babel">get started with Babel</a>)</em> in the React article series I&#x2019;ve been writing about, and now I want to <strong>focus on the language features</strong> themselves. I&#x2019;ve read a <em>ton</em> about ES6 and ES7 and it&#x2019;s about time we started discussing ES6 and ES7 features here in Pony Foo.</p></div>

<div></div>

<div><p>This article <strong>warns about going overboard</strong> with ES6 language features. Then we&#x2019;ll start off the series by discussing about Destructuring in ES6, and when it&#x2019;s most useful, as well as some of its gotchas and caveats.</p> <h2 id="a-word-of-caution">A word of caution</h2> <p>When uncertain, chances are <mark class="md-mark">you probably should default to ES5 and older syntax instead of adopting ES6 just because you can</mark>. By this I don&#x2019;t mean that using ES6 syntax is a bad idea &#x2013; quite the opposite, see I&#x2019;m writing an article about ES6! My concern lies with the fact that when we adopt ES6 features we must do it because <strong>they&#x2019;ll absolutely improve our code quality</strong>, and not just because of the <em>&#x201C;cool factor&#x201D;</em> &#x2013; whatever that may be.</p> <p>The approach I&#x2019;ve been taking thus far is to write things in <em>plain ES5</em>, and then adding ES6 sugar on top where it&#x2019;d genuinely improve my code. I presume over time I&#x2019;ll be able to more quickly identify scenarios where a ES6 feature may be worth using over ES5, but when getting started it might be a good idea <em>not</em> to go overboard too soon. Instead, carefully analyze what would fit your code best first, and <strong>be mindful of adopting ES6</strong>.</p> <blockquote> <p>This way, you&#x2019;ll <strong>learn to use</strong> the new features in your favor, rather than just <em>learning the syntax</em>.</p> </blockquote> <p>Onto the cool stuff now!</p></div>

<div><h2 id="destructuring">Destructuring</h2> <p>This is easily one of the features I&#x2019;ve been using the most. It&#x2019;s also one of the simplest. It binds properties to as many variables as you need and it works with both Arrays and Objects.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = { bar: <span class="md-code-string">&apos;pony&apos;</span>, baz: <span class="md-code-number">3</span> }
<span class="md-code-keyword">var</span> {bar, baz} = foo
<span class="md-code-built_in">console</span>.log(bar)
<span class="md-code-comment">// &lt;- &apos;pony&apos;</span>
<span class="md-code-built_in">console</span>.log(baz)
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>It makes it very quick to pull out a specific property from an object. You&#x2019;re also allowed to map properties into aliases as well.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = { bar: <span class="md-code-string">&apos;pony&apos;</span>, baz: <span class="md-code-number">3</span> }
<span class="md-code-keyword">var</span> {bar: a, baz: b} = foo
<span class="md-code-built_in">console</span>.log(a)
<span class="md-code-comment">// &lt;- &apos;pony&apos;</span>
<span class="md-code-built_in">console</span>.log(b)
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>You can also pull properties as deep as you want, and you could also alias those deep bindings.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = { bar: { deep: <span class="md-code-string">&apos;pony&apos;</span>, dangerouslySetInnerHTML: <span class="md-code-string">&apos;lol&apos;</span> } }
<span class="md-code-keyword">var</span> {bar: { deep, dangerouslySetInnerHTML: sure }} = foo
<span class="md-code-built_in">console</span>.log(deep)
<span class="md-code-comment">// &lt;- &apos;pony&apos;</span>
<span class="md-code-built_in">console</span>.log(sure)
<span class="md-code-comment">// &lt;- &apos;lol&apos;</span>
</code></pre> <p>By default, properties that aren&#x2019;t found will be <code class="md-code md-code-inline">undefined</code>, just like when accessing properties on an object with the dot or bracket notation.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> {foo} = {bar: <span class="md-code-string">&apos;baz&apos;</span>}
<span class="md-code-built_in">console</span>.log(foo)
<span class="md-code-comment">// &lt;- undefined</span>
</code></pre> <p>If you&#x2019;re trying to access a deeply nested property of a parent that doesn&#x2019;t exist, then you&#x2019;ll get an exception, though.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> {foo:{bar}} = {baz: <span class="md-code-string">&apos;ouch&apos;</span>}
<span class="md-code-comment">// &lt;- Exception</span>
</code></pre> <p>That makes a lot of sense, if you think of destructuring as sugar for ES5 like the code below.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> _temp = { baz: <span class="md-code-string">&apos;ouch&apos;</span> }
<span class="md-code-keyword">var</span> bar = _temp.foo.bar
<span class="md-code-comment">// &lt;- Exception</span>
</code></pre> <p>A cool property of destructuring is that it allows you to swap variables without the need for the infamous <code class="md-code md-code-inline">aux</code> variable.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">es5</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> left = <span class="md-code-number">10</span>
  <span class="md-code-keyword">var</span> right = <span class="md-code-number">20</span>
  <span class="md-code-keyword">var</span> aux
  <span class="md-code-keyword">if</span> (right &gt; left) {
    aux = right
    right = left
    left = aux
  }
}
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">es6</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> left = <span class="md-code-number">10</span>
  <span class="md-code-keyword">var</span> right = <span class="md-code-number">20</span>
  <span class="md-code-keyword">if</span> (right &gt; left) {
    <mark class="md-mark md-code-mark">[left, right] = [right, left]</mark>
  }
}
</code></pre> <p>Another convenient aspect of destructuring is the ability to pull keys using <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names" target="_blank" aria-label="Computed Property Names &#x2013; MDN">computed property names</a>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> key = <span class="md-code-string">&apos;such_dynamic&apos;</span>
<span class="md-code-keyword">var</span> { <mark class="md-mark md-code-mark">[key]</mark>: foo } = { such_dynamic: <span class="md-code-string">&apos;bar&apos;</span> }
<span class="md-code-built_in">console</span>.log(foo)
<span class="md-code-comment">// &lt;- &apos;bar&apos;</span>
</code></pre> <p>In ES5, that&#x2019;d take an extra statement and variable allocation on your behalf.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> key = <span class="md-code-string">&apos;such_dynamic&apos;</span>
<span class="md-code-keyword">var</span> baz = { such_dynamic: <span class="md-code-string">&apos;bar&apos;</span> }
<span class="md-code-keyword">var</span> foo = baz[key]
<span class="md-code-built_in">console</span>.log(foo)
</code></pre> <p>You can also define default values, for the case where the pulled property evaluates to <code class="md-code md-code-inline">undefined</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> {foo=<span class="md-code-number">3</span>} = { foo: <span class="md-code-number">2</span> }
<span class="md-code-built_in">console</span>.log(foo)
<span class="md-code-comment">// &lt;- 2</span>
<span class="md-code-keyword">var</span> {foo=<span class="md-code-number">3</span>} = { foo: <span class="md-code-literal">undefined</span> }
<span class="md-code-built_in">console</span>.log(foo)
<span class="md-code-comment">// &lt;- 3</span>
<span class="md-code-keyword">var</span> {foo=<span class="md-code-number">3</span>} = { bar: <span class="md-code-number">2</span> }
<span class="md-code-built_in">console</span>.log(foo)
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>Destructuring works for Arrays as well, as we mentioned earlier. Note how I&#x2019;m <strong>using square brackets</strong> in the destructuring side of the declaration now.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> [a] = [<span class="md-code-number">10</span>]
<span class="md-code-built_in">console</span>.log(a)
<span class="md-code-comment">// &lt;- 10</span>
</code></pre> <p>Here, again, we can use the default values and follow the same rules.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> [a] = []
<span class="md-code-built_in">console</span>.log(a)
<span class="md-code-comment">// &lt;- undefined</span>
<span class="md-code-keyword">var</span> [b=<span class="md-code-number">10</span>] = [<span class="md-code-literal">undefined</span>]
<span class="md-code-built_in">console</span>.log(b)
<span class="md-code-comment">// &lt;- 10</span>
<span class="md-code-keyword">var</span> [c=<span class="md-code-number">10</span>] = []
<span class="md-code-built_in">console</span>.log(c)
<span class="md-code-comment">// &lt;- 10</span>
</code></pre> <p>When it comes to Arrays you can conveniently skip over elements that you don&#x2019;t care about.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> [,,a,b] = [<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>,<span class="md-code-number">4</span>,<span class="md-code-number">5</span>]
<span class="md-code-built_in">console</span>.log(a)
<span class="md-code-comment">// &lt;- 3</span>
<span class="md-code-built_in">console</span>.log(b)
<span class="md-code-comment">// &lt;- 4</span>
</code></pre> <p>You can also use destructuring in a <code class="md-code md-code-inline">function</code>&apos;s parameter list.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">function greet ({ age, name:greeting=&apos;she&apos; }) {
  console.log(<mark class="md-mark md-code-mark">`${greeting} is ${age} years old.`</mark>)
}
greet({ name: &apos;nico&apos;, age: 27 })
// &lt;- &apos;nico is 27 years old&apos;
greet({ age: 24 })
// &lt;- &apos;she is 24 years old&apos;
</code></pre> <p>That&#x2019;s roughly <strong>how</strong> you can use destructuring. What is destructuring <strong>good</strong> for?</p> <h2 id="use-cases-for-destructuring">Use Cases for Destructuring</h2> <p>There are many situations where destructuring comes in handy. Here&#x2019;s some of the most common ones. Whenever you have a method that returns an object, destructuring makes it much terser to interact with.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">getCoords</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> {
    x: <span class="md-code-number">10</span>,
    y: <span class="md-code-number">22</span>
  }
}
<span class="md-code-keyword">var</span> {x, y} = getCoords()
<span class="md-code-built_in">console</span>.log(x)
<span class="md-code-comment">// &lt;- 10</span>
<span class="md-code-built_in">console</span>.log(y)
<span class="md-code-comment">// &lt;- 22</span>
</code></pre> <p>A similar use case but in the opposite direction is being able to define default options when you have a method with a bunch of options that need default values. This is particularly interesting as an alternative to named parameters in other languages like Python and C#.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">random</span> <span class="md-code-params">({ min=1, max=300 })</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-built_in">Math</span>.floor(<span class="md-code-built_in">Math</span>.random() * (max - min)) + min
}
<span class="md-code-built_in">console</span>.log(random({}))
<span class="md-code-comment">// &lt;- 174</span>
<span class="md-code-built_in">console</span>.log(random({max: <span class="md-code-number">24</span>}))
<span class="md-code-comment">// &lt;- 18</span>
</code></pre> <p>If you wanted to make the options object <em>entirely optional</em> you could change the syntax to the following.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">random</span> <span class="md-code-params">({ min=1, max=300 }<mark class="md-mark md-code-mark"> = {}</mark>)</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-built_in">Math</span>.floor(<span class="md-code-built_in">Math</span>.random() * (max - min)) + min
}
<span class="md-code-built_in">console</span>.log(random())
<span class="md-code-comment">// &lt;- 133</span>
</code></pre> <p>A great fit for destructuring are things like regular expressions, where you would just love to name parameters without having to resort to index numbers. Here&#x2019;s an example parsing a URL with a random <code class="md-code md-code-inline">RegExp</code> <a href="http://stackoverflow.com/a/27755/389745" target="_blank" aria-label="Getting parts of a URL on StackOverflow">I got on StackOverflow</a>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">getUrlParts</span> <span class="md-code-params">(url)</span> </span>{
  <span class="md-code-keyword">var</span> magic = <span class="md-code-regexp">/^(https?):\/\/(ponyfoo\.com)(\/articles\/([a-z0-9-]+))$/</span>
  <span class="md-code-keyword">return</span> magic.exec(url)
}
<span class="md-code-keyword">var</span> parts = getUrlParts(<span class="md-code-string">&apos;http://ponyfoo.com/articles/es6-destructuring-in-depth&apos;</span>)
<span class="md-code-keyword">var</span> [,protocol,host,pathname,slug] = parts
<span class="md-code-built_in">console</span>.log(protocol)
<span class="md-code-comment">// &lt;- &apos;http&apos;</span>
<span class="md-code-built_in">console</span>.log(host)
<span class="md-code-comment">// &lt;- &apos;ponyfoo.com&apos;</span>
<span class="md-code-built_in">console</span>.log(pathname)
<span class="md-code-comment">// &lt;- &apos;/articles/es6-destructuring-in-depth&apos;</span>
<span class="md-code-built_in">console</span>.log(slug)
<span class="md-code-comment">// &lt;- &apos;es6-destructuring-in-depth&apos;</span>
</code></pre> <h3 id="special-case-import-statements">Special Case: <code class="md-code md-code-inline">import</code> Statements</h3> <p>Even though <code class="md-code md-code-inline">import</code> statements don&#x2019;t follow destructuring rules, they behave a bit similarly. This is probably the <em>&#x201C;destructuring-like&#x201D;</em> use case I find myself using the most, even though it&#x2019;s not actually destructuring. Whenever you&#x2019;re writing module <code class="md-code md-code-inline">import</code> statements, you can pull just what you need from a module&#x2019;s public API. An example using <a href="https://github.com/bevacqua/contra" target="_blank" aria-label="bevacqua/contra on GitHub"><code class="md-code md-code-inline">contra</code></a>:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">import {series, concurrent, map } from <span class="md-code-string">&apos;contra&apos;</span>
series(tasks, done)
concurrent(tasks, done)
map(items, mapper, done)
</code></pre> <p>Note that, however, <code class="md-code md-code-inline">import</code> statements have a different syntax. When compared against destructuring, none of the following <code class="md-code md-code-inline">import</code> statements will work.</p> <ul> <li>Use defaults values such as <code class="md-code md-code-inline">import {series = noop} from &apos;contra&apos;</code></li> <li>&#x201C;Deep&#x201D; destructuring style like <code class="md-code md-code-inline">import {map: { series }} from &apos;contra&apos;</code></li> <li>Aliasing syntax <code class="md-code md-code-inline">import {map: mapAsync} from &apos;contra&apos;</code></li> </ul> <p>The main reason for these limitations is that the <code class="md-code md-code-inline">import</code> statement brings in a <em>binding</em>, and not a reference or a value. This is an important differentiation that we&#x2019;ll explore more in depth in a future article about ES6 modules.</p> <blockquote> <p>I&#x2019;ll keep posting about ES6 &amp; ES7 features every day, so make sure to subscribe if you want to know more!</p> </blockquote> <p><sub><mark class="md-mark">*</mark> How about we visit string interpolation tomorrow?</sub><br> <sub><mark class="md-mark">**</mark>We&#x2019;ll leave arrow functions for monday!</sub></p></div>
