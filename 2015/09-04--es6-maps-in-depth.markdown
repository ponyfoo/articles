<h1>ES6 Maps in Depth</h1>

<div><kbd>es6</kbd> <kbd>maps</kbd> <kbd>es6-in-depth</kbd></div>

<blockquote><p>Hello, this is <a href="https://ponyfoo.com/articles/tagged/es6-in-depth">ES6 &#x2013; <em>&#x201C;Please make them stop&#x201D;</em> &#x2013; in Depth</a>. New here? Start with <a href="https://ponyfoo.com/articles/a-brief-history-of-es6-tooling">A Brief History of ES6 Tooling</a>. Then, make your way through <a href="https://ponyfoo.com/articles/es6-destructuring-in-depth">&#x2026;</a></p></blockquote>

<div><p>Hello, this is <a href="https://ponyfoo.com/articles/tagged/es6-in-depth">ES6 &#x2013; <em>&#x201C;Please make them stop&#x201D;</em> &#x2013; in Depth</a>. New here? Start with <a href="https://ponyfoo.com/articles/a-brief-history-of-es6-tooling">A Brief History of ES6 Tooling</a>. Then, make your way through <a href="https://ponyfoo.com/articles/es6-destructuring-in-depth">destructuring</a>, <a href="https://ponyfoo.com/articles/es6-template-strings-in-depth">template literals</a>, <a href="https://ponyfoo.com/articles/es6-arrow-functions-in-depth">arrow functions</a>, the <a href="https://ponyfoo.com/articles/es6-spread-and-butter-in-depth">spread operator and rest parameters</a>, improvements coming to <a href="https://ponyfoo.com/articles/es6-object-literal-features-in-depth">object literals</a>, the new <a href="https://ponyfoo.com/articles/es6-classes-in-depth"><em>classes</em></a> sugar on top of prototypes, <a href="https://ponyfoo.com/articles/es6-let-const-and-temporal-dead-zone-in-depth"><code class="md-code md-code-inline">let</code>, <code class="md-code md-code-inline">const</code>, and the <em>&#x201C;Temporal Dead Zone&#x201D;</em></a>, <a href="https://ponyfoo.com/articles/es6-iterators-in-depth">iterators</a>, <a href="https://ponyfoo.com/articles/es6-generators-in-depth">generators</a>, and <a href="https://ponyfoo.com/articles/es6-symbols-in-depth">Symbols</a>. Today we&#x2019;ll be discussing a new collection data structure objects coming in ES6 &#x2013; I&#x2019;m talking about <code class="md-code md-code-inline">Map</code>.</p></div>

<div></div>

<div><blockquote> <p>Like I did in previous articles on the series, I would love to point out that you should probably <a href="https://ponyfoo.com/articles/universal-react-babel#setting-up-babel">set up Babel</a> and follow along the examples with either a REPL or the <code class="md-code md-code-inline">babel-node</code> CLI and a file. That&#x2019;ll make it so much easier for you to <strong>internalize the concepts</strong> discussed in the series. If you aren&#x2019;t the <em>&#x201C;install things on my computer&#x201D;</em> kind of human, you might prefer to hop on <a href="http://codepen.io/" target="_blank">CodePen</a> and then click on the gear icon for JavaScript &#x2013; <em>they have a Babel preprocessor which makes trying out ES6 a breeze.</em> Another alternative that&#x2019;s also quite useful is to use Babel&#x2019;s <a href="http://babeljs.io/repl/" target="_blank">online REPL</a> <em>&#x2013; it&#x2019;ll show you compiled ES5 code to the right of your ES6 code for quick comparison.</em></p> </blockquote> <p>Before getting into it, let me <a href="https://www.patreon.com/bevacqua" target="_blank"><em>shamelessly ask for your support</em></a> if you&#x2019;re enjoying my <a href="https://ponyfoo.com/articles/tagged/es6-in-depth">ES6 in Depth</a> series. Your contributions will go towards helping me keep up with the schedule, server bills, keeping me fed, and maintaining <strong>Pony Foo</strong> as a veritable source of JavaScript goodies.</p> <p>Thanks for reading that, and let&#x2019;s go into collections now! For a bit of context, you may want to check out the article on <a href="https://ponyfoo.com/articles/es6-iterators-in-depth">iterators</a> <em>&#x2013; which are closely related to ES6 collections &#x2013;</em> and the one on <a href="https://ponyfoo.com/articles/es6-spread-and-butter-in-depth">spread and rest parameters</a>.</p> <p>Now, let&#x2019;s start with <code class="md-code md-code-inline">Map</code>. I moved the rest of the ES6 collections to tomorrow&#x2019;s publication in order to keep the series sane, as otherwise this would&#x2019;ve been too long for a single article!</p></div>

<div><h1 id="before-es6-there-were-hash-maps">Before ES6, There Were Hash-Maps</h1> <p>A very common <em>ab</em>use case of JavaScript objects is hash-maps, where we map string keys to arbitrary values. For example, one might use an object to map <code class="md-code md-code-inline">npm</code> package names to their metadata, like so:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> registry = {}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">add</span> <span class="md-code-params">(name, meta)</span> </span>{
  registry[name] = meta
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">get</span> <span class="md-code-params">(name)</span> </span>{
  <span class="md-code-keyword">return</span> registry[name]
}
add(<span class="md-code-string">&apos;contra&apos;</span>, { description: <span class="md-code-string">&apos;Asynchronous flow control&apos;</span> })
add(<span class="md-code-string">&apos;dragula&apos;</span>, { description: <span class="md-code-string">&apos;Drag and drop&apos;</span> })
add(<span class="md-code-string">&apos;woofmark&apos;</span>, { description: <span class="md-code-string">&apos;Markdown and WYSIWYG editor&apos;</span> })
</code></pre> <p>There&#x2019;s several issues with this approach, to wit:</p> <ul> <li><strong>Security issues</strong> where user-provided keys like <code class="md-code md-code-inline">__proto__</code>, <code class="md-code md-code-inline">toString</code>, or anything in <code class="md-code md-code-inline">Object.prototype</code> break expectations and make interaction with these kinds of <em>hash-map</em> data structures more cumbersome</li> <li>Iteration over list items is verbose with <code class="md-code md-code-inline">Object.keys(registry).forEach</code> or implementing the <a href="https://ponyfoo.com/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo"><em>iterable</em> protocol</a> on the <code class="md-code md-code-inline">registry</code></li> <li>Keys are limited to strings, making it hard to create hash-maps where you&#x2019;d like to index values by DOM elements or other non-string references</li> </ul> <p>The first problem could be fixed using a prefix, and being careful to always get or set values in the hash-map through methods. It would be even better to use <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Proxy" target="_blank" aria-label="Proxy Objects in ES6 on MDN">ES6 proxies</a>, but we <em>won&#x2019;t be covering those until tomorrow!</em></p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> registry = {}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">add</span> <span class="md-code-params">(name, meta)</span> </span>{
  registry[<mark class="md-mark md-code-mark"><span class="md-code-string">&apos;map:&apos;</span> + </mark>name] = meta
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">get</span> <span class="md-code-params">(name)</span> </span>{
  <span class="md-code-keyword">return</span> registry[<mark class="md-mark md-code-mark"><span class="md-code-string">&apos;map:&apos;</span> + </mark>name]
}
add(<span class="md-code-string">&apos;contra&apos;</span>, { description: <span class="md-code-string">&apos;Asynchronous flow control&apos;</span> })
add(<span class="md-code-string">&apos;dragula&apos;</span>, { description: <span class="md-code-string">&apos;Drag and drop&apos;</span> })
add(<span class="md-code-string">&apos;woofmark&apos;</span>, { description: <span class="md-code-string">&apos;Markdown and WYSIWYG editor&apos;</span> })
</code></pre> <p>Luckily for us, though, <em>ES6 maps</em> provide us with an even better solution to the key-naming security issue. At the same time they facilitate collection behaviors out the box that may also come in handy. Let&#x2019;s plunge into their practical usage and inner workings.</p> <h1 id="es6-maps">ES6 Maps</h1> <p>Map is a key/value data structure in ES6. It provides a better data structure to be used for hash-maps. Here&#x2019;s how what we had earlier looks like with ES6 maps.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <mark class="md-mark md-code-mark">new Map()</mark>
<mark class="md-mark md-code-mark">map.set</mark>(<span class="md-code-string">&apos;contra&apos;</span>, { description: <span class="md-code-string">&apos;Asynchronous flow control&apos;</span> })
map.set(<span class="md-code-string">&apos;dragula&apos;</span>, { description: <span class="md-code-string">&apos;Drag and drop&apos;</span> })
map.set(<span class="md-code-string">&apos;woofmark&apos;</span>, { description: <span class="md-code-string">&apos;Markdown and WYSIWYG editor&apos;</span> })
</code></pre> <p>One of the important differences is also that you&#x2019;re able to use anything for the keys. You&#x2019;re not just limited to primitive values like symbols, numbers, or strings, but you can even use functions, objects and dates &#x2013; too. Keys won&#x2019;t be casted to strings like with regular objects, either.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
map.set(<span class="md-code-keyword">new</span> <span class="md-code-built_in">Date</span>(), <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">today</span> <span class="md-code-params">()</span> </span>{})
map.set(() =&gt; <span class="md-code-string">&apos;key&apos;</span>, { pony: <span class="md-code-string">&apos;foo&apos;</span> })
map.set(Symbol(<span class="md-code-string">&apos;items&apos;</span>), [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>])
</code></pre> <p>You can also provide <code class="md-code md-code-inline">Map</code> objects with any object that follows the <a href="https://ponyfoo.com/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo"><em>iterable</em> protocol</a> and produces a collection such as <code class="md-code md-code-inline">[[&apos;key&apos;, &apos;value&apos;], [&apos;key&apos;, &apos;value&apos;]]</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([
  [<span class="md-code-keyword">new</span> <span class="md-code-built_in">Date</span>(), <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">today</span> <span class="md-code-params">()</span> </span>{}],
  [() =&gt; <span class="md-code-string">&apos;key&apos;</span>, { pony: <span class="md-code-string">&apos;foo&apos;</span> }],
  [Symbol(<span class="md-code-string">&apos;items&apos;</span>), [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>]]
])
</code></pre> <p>The above would be effectively the same as the following. Note how we&#x2019;re using destructuring in the parameters of <code class="md-code md-code-inline">items.forEach</code> to <em>effortlessly</em> pull the <code class="md-code md-code-inline">key</code> and <code class="md-code md-code-inline">value</code> out of the two-dimensional <code class="md-code md-code-inline">item</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> items = [
  [<span class="md-code-keyword">new</span> <span class="md-code-built_in">Date</span>(), <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">today</span> <span class="md-code-params">()</span> </span>{}],
  [() =&gt; <span class="md-code-string">&apos;key&apos;</span>, { pony: <span class="md-code-string">&apos;foo&apos;</span> }],
  [Symbol(<span class="md-code-string">&apos;items&apos;</span>), [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>]]
]
<span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
items.forEach((<mark class="md-mark md-code-mark">[key, value]</mark>) =&gt; map.set(key, value))
</code></pre> <p>Of course, it&#x2019;s kind of silly to go through the trouble of adding items one by one when you can just feed an iterable to your <code class="md-code md-code-inline">Map</code>. Speaking of iterables &#x2013; <code class="md-code md-code-inline">Map</code> adheres to the <a href="https://ponyfoo.com/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo"><em>iterable</em></a> protocol. It&#x2019;s very easy to pull a key-value pair collection much like the ones you can feed to the <code class="md-code md-code-inline">Map</code> constructor.</p> <p>Naturally, we can use the spread operator to this effect.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
map.set(<span class="md-code-string">&apos;p&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>)
map.set(<span class="md-code-string">&apos;n&apos;</span>, <span class="md-code-string">&apos;y&apos;</span>)
map.set(<span class="md-code-string">&apos;f&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>)
map.set(<span class="md-code-string">&apos;o&apos;</span>, <span class="md-code-string">&apos;!&apos;</span>)
<span class="md-code-built_in">console</span>.log([...map])
<span class="md-code-comment">// &lt;- [[&apos;p&apos;, &apos;o&apos;], [&apos;n&apos;, &apos;y&apos;], [&apos;f&apos;, &apos;o&apos;], [&apos;o&apos;, &apos;!&apos;]]</span>
</code></pre> <p>You could also use a <code class="md-code md-code-inline">for..of</code> loop, and we could combine that with <a href="https://ponyfoo.com/articles/es6-destructuring-in-depth" aria-label="ES6 JavaScript Destructuring in Depth on Pony Foo">destructuring</a> to make it seriously terse. Also, remember <a href="https://ponyfoo.com/articles/es6-template-strings-in-depth" aria-label="ES6 Template Literals in Depth on Pony Foo">template literals</a>?</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
map.set(<span class="md-code-string">&apos;p&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>)
map.set(<span class="md-code-string">&apos;n&apos;</span>, <span class="md-code-string">&apos;y&apos;</span>)
map.set(<span class="md-code-string">&apos;f&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>)
map.set(<span class="md-code-string">&apos;o&apos;</span>, <span class="md-code-string">&apos;!&apos;</span>)
<span class="md-code-keyword">for</span> (<span class="md-code-keyword">let</span> <mark class="md-mark md-code-mark">[key, value]</mark> of map) {
  <span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">`${key}: ${value}`</mark>)
  <span class="md-code-comment">// &lt;- &apos;p: o&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;n: y&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;f: o&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;o: !&apos;</span>
}
</code></pre> <p>Even though maps have a programmatic API to add items, keys are unique, just like with hash-maps. Setting a key over and over again will only overwrite its value.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
map.set(<span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-string">&apos;a&apos;</span>)
map.set(<span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-string">&apos;b&apos;</span>)
map.set(<span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-string">&apos;c&apos;</span>)
<span class="md-code-built_in">console</span>.log([...map])
<span class="md-code-comment">// &lt;- [[&apos;a&apos;, &apos;c&apos;]]</span>
</code></pre> <p>In ES6 <code class="md-code md-code-inline">Map</code>, <code class="md-code md-code-inline">NaN</code> becomes a &#x201C;corner-case&#x201D; that gets <strong>treated as a value that&#x2019;s equal to itself</strong> even though the following expression actually evaluates to <code class="md-code md-code-inline">true</code> &#x2013; <code class="md-code md-code-inline">NaN !== NaN</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">console</span>.log(<span class="md-code-literal">NaN</span> === <span class="md-code-literal">NaN</span>)
<span class="md-code-comment">// &lt;- false</span>
<span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map()
map.set(<span class="md-code-literal">NaN</span>, <span class="md-code-string">&apos;foo&apos;</span>)
map.set(<span class="md-code-literal">NaN</span>, <span class="md-code-string">&apos;bar&apos;</span>)
<span class="md-code-built_in">console</span>.log([...map])
<span class="md-code-comment">// &lt;- [[NaN, &apos;bar&apos;]]</span>
</code></pre> <h2 id="hash-maps-and-the-dom">Hash-Maps and the DOM</h2> <p>In ES5, whenever we had a DOM element we wanted to associate with an API object for some library, we had to follow a verbose and slow pattern like the one below. The following piece of code just returns an API object with a bunch of methods for a given DOM element, allowing us to put and remove DOM elements from the cache, and also allowing us to retrieve the API object for a DOM element &#x2013; if one already exists.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> cache = []
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">put</span> <span class="md-code-params">(el, api)</span> </span>{
  cache.push({ el: el, api: api })
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">find</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">for</span> (i = <span class="md-code-number">0</span>; i &lt; cache.length; i++) {
    <span class="md-code-keyword">if</span> (cache[i].el === el) {
      <span class="md-code-keyword">return</span> cache[i].api
    }
  }
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">destroy</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">for</span> (i = <span class="md-code-number">0</span>; i &lt; cache.length; i++) {
    <span class="md-code-keyword">if</span> (cache[i].el === el) {
      cache.splice(i, <span class="md-code-number">1</span>)
      <span class="md-code-keyword">return</span>
    }
  }
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">thing</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">var</span> api = find(el)
  <span class="md-code-keyword">if</span> (api) {
    <span class="md-code-keyword">return</span> api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: destroy.bind(<span class="md-code-literal">null</span>, el)
  }
  put(el, api)
  <span class="md-code-keyword">return</span> api
}
</code></pre> <p>One of the coolest aspects of <code class="md-code md-code-inline">Map</code>, <em>as I&#x2019;ve previously mentioned</em>, is the ability to index by DOM elements. The fact that <code class="md-code md-code-inline">Map</code> also has collection manipulation abilities also greatly simplifies things.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> cache = <span class="md-code-keyword">new</span> Map()
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">put</span> <span class="md-code-params">(el, api)</span> </span>{
  <mark class="md-mark md-code-mark">cache.set(el, api)</mark>
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">find</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">return</span> <mark class="md-mark md-code-mark">cache.get(el)</mark>
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">destroy</span> <span class="md-code-params">(el)</span> </span>{
  <mark class="md-mark md-code-mark">cache.delete(el)</mark>
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">thing</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">var</span> api = find(el)
  <span class="md-code-keyword">if</span> (api) {
    <span class="md-code-keyword">return</span> api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: destroy.bind(<span class="md-code-literal">null</span>, el)
  }
  put(el, api)
  <span class="md-code-keyword">return</span> api
}
</code></pre> <p>The fact that these methods have now become one liners means we can just inline them, as readability is no longer an issue. We just went from <em>~30 LOC</em> to <strong>half that amount</strong>. Needless to say, at some point in the future this will also perform <em>much</em> faster than the haystack alternative.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> cache = <span class="md-code-keyword">new</span> Map()
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">thing</span> <span class="md-code-params">(el)</span> </span>{
  <span class="md-code-keyword">var</span> api = <mark class="md-mark md-code-mark">cache.get(el)</mark>
  <span class="md-code-keyword">if</span> (api) {
    <span class="md-code-keyword">return</span> api
  }
  api = {
    method: method,
    method2: method2,
    method3: method3,
    destroy: () =&gt; <mark class="md-mark md-code-mark">cache.delete(el)</mark>
  }
  <mark class="md-mark md-code-mark">cache.set(el, api)</mark>
  <span class="md-code-keyword">return</span> api
}
</code></pre> <p>The simplicity of <code class="md-code md-code-inline">Map</code> is amazing. If you ask me, we desperately needed this feature in JavaScript. Being to index a collection by arbitrary objects is <strong>super important</strong>.</p> <blockquote> <p>What else can we do with <code class="md-code md-code-inline">Map</code>?</p> </blockquote> <h2 id="collection-methods-in-map">Collection Methods in <code class="md-code md-code-inline">Map</code></h2> <p>Maps make it very easy to probe the collection and figure out whether a <code class="md-code md-code-inline">key</code> is defined in the <code class="md-code md-code-inline">Map</code>. As we noted earlier, <code class="md-code md-code-inline">NaN</code> equals <code class="md-code md-code-inline">NaN</code> as far as <code class="md-code md-code-inline">Map</code> is concerned. However, <code class="md-code md-code-inline">Symbol</code> values are always different, so you&#x2019;ll have to use them by value!</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-literal">NaN</span>, <span class="md-code-number">1</span>], [Symbol(), <span class="md-code-number">2</span>], [<span class="md-code-string">&apos;foo&apos;</span>, <span class="md-code-string">&apos;bar&apos;</span>]])
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-literal">NaN</span>))
<span class="md-code-comment">// &lt;- true</span>
<span class="md-code-built_in">console</span>.log(map.has(Symbol()))
<span class="md-code-comment">// &lt;- <mark class="md-mark md-code-mark">false</mark></span>
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-string">&apos;foo&apos;</span>))
<span class="md-code-comment">// &lt;- true</span>
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-string">&apos;bar&apos;</span>))
<span class="md-code-comment">// &lt;- false</span>
</code></pre> <p>As long as you keep a <code class="md-code md-code-inline">Symbol</code> reference around, you&#x2019;ll be okay. <em>Keep your references close, and your <code class="md-code md-code-inline">Symbol</code>s closer?</em></p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><mark class="md-mark md-code-mark">var sym = Symbol()</mark>
<span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-literal">NaN</span>, <span class="md-code-number">1</span>], [sym, <span class="md-code-number">2</span>], [<span class="md-code-string">&apos;foo&apos;</span>, <span class="md-code-string">&apos;bar&apos;</span>]])
<span class="md-code-built_in">console</span>.log(map.has(sym))
<span class="md-code-comment">// &lt;- <mark class="md-mark md-code-mark">true</mark></span>
</code></pre> <p>Also, remember the <strong>no key-casting</strong> thing? <em>Beware!</em> We are so used to objects casting keys to strings that this may bite you if you&#x2019;re not careful.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-number">1</span>, <span class="md-code-string">&apos;a&apos;</span>]])
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-number">1</span>))
<span class="md-code-comment">// &lt;- true</span>
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-string">&apos;1&apos;</span>))
<span class="md-code-comment">// &lt;- <mark class="md-mark md-code-mark">false</mark></span>
</code></pre> <p>You can also clear a <code class="md-code md-code-inline">Map</code> entirely of entries without losing a reference to it. This can be very handy sometimes.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-number">1</span>, <span class="md-code-number">2</span>], [<span class="md-code-number">3</span>, <span class="md-code-number">4</span>], [<span class="md-code-number">5</span>, <span class="md-code-number">6</span>]])
map.clear()
<span class="md-code-built_in">console</span>.log(map.has(<span class="md-code-number">1</span>))
<span class="md-code-comment">// &lt;- false</span>
<span class="md-code-built_in">console</span>.log([...map])
<span class="md-code-comment">// &lt;- []</span>
</code></pre> <p>When you use <code class="md-code md-code-inline">Map</code> as an iterable, you are actually looping over its <code class="md-code md-code-inline">.entries()</code>. That means that you don&#x2019;t need to <strong>explicitly</strong> iterate over <code class="md-code md-code-inline">.entries()</code>. It&#x2019;ll be done on your behalf anyways. You do remember <a href="https://ponyfoo.com/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo"><code class="md-code md-code-inline">Symbol.iterator</code></a>, right?</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">console</span>.log(map[<mark class="md-mark md-code-mark">Symbol.iterator</mark>] === map.entries)
<span class="md-code-comment">// &lt;- true</span>
</code></pre> <p>Just like <code class="md-code md-code-inline">.entries()</code>, <code class="md-code md-code-inline">Map</code> has two other iterators you can leverage. These are <code class="md-code md-code-inline">.keys()</code> and <code class="md-code md-code-inline">.values()</code>. I&#x2019;m sure you guessed what sequences of values they yield, but here&#x2019;s a code snippet anyways.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-number">1</span>, <span class="md-code-number">2</span>], [<span class="md-code-number">3</span>, <span class="md-code-number">4</span>], [<span class="md-code-number">5</span>, <span class="md-code-number">6</span>]])
<span class="md-code-built_in">console</span>.log([...map.keys()])
<span class="md-code-comment">// &lt;- [1, 3, 5]</span>
<span class="md-code-built_in">console</span>.log([...map.values()])
<span class="md-code-comment">// &lt;- [2, 4, 6]</span>
</code></pre> <p>Maps also come with a <em>read-only</em> <code class="md-code md-code-inline">.size</code> property that behaves sort of like <code class="md-code md-code-inline">Array.prototype.length</code> &#x2013; at any point in time it gives you the current amount of entries in the map.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-number">1</span>, <span class="md-code-number">2</span>], [<span class="md-code-number">3</span>, <span class="md-code-number">4</span>], [<span class="md-code-number">5</span>, <span class="md-code-number">6</span>]])
<span class="md-code-built_in">console</span>.log(map.size)
<span class="md-code-comment">// &lt;- 3</span>
map.delete(<span class="md-code-number">3</span>)
<span class="md-code-built_in">console</span>.log(map.size)
<span class="md-code-comment">// &lt;- 2</span>
map.clear()
<span class="md-code-built_in">console</span>.log(map.size)
<span class="md-code-comment">// &lt;- 0</span>
</code></pre> <p>One more aspect of <code class="md-code md-code-inline">Map</code> that&#x2019;s worth mentioning is that their entries are always iterated in <strong>insertion order</strong>. This is in contrast with <code class="md-code md-code-inline">Object.keys</code> loops which follow <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in" target="_blank" aria-label="for..in on MDN">an arbitrary order</a>.</p> <blockquote> <p>The <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in" target="_blank" aria-label="for..in on MDN"><code class="md-code md-code-inline">for..in</code></a> statement iterates over the enumerable properties of an object, in arbitrary order.</p> </blockquote> <p>Maps also have a <code class="md-code md-code-inline">.forEach</code> method that&#x2019;s identical in <em>behavior</em> to that in ES5 <code class="md-code md-code-inline">Array</code> objects. Once again, keys do not get casted into strings here.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> map = <span class="md-code-keyword">new</span> Map([[<span class="md-code-literal">NaN</span>, <span class="md-code-number">1</span>], [Symbol(), <span class="md-code-number">2</span>], [<span class="md-code-string">&apos;foo&apos;</span>, <span class="md-code-string">&apos;bar&apos;</span>]])
map.forEach(<mark class="md-mark md-code-mark">(value, key)</mark> =&gt; <span class="md-code-built_in">console</span>.log(key, value))
<span class="md-code-comment">// &lt;- NaN 1</span>
<span class="md-code-comment">// &lt;- Symbol() 2</span>
<span class="md-code-comment">// &lt;- &apos;foo&apos; &apos;bar&apos;</span>
</code></pre> <blockquote> <p>Get up early tomorrow morning, we&#x2019;ll be having <a href="http://ponyfoo.com/articles/es6-weakmaps-sets-and-weaksets-in-depth" target="_blank" aria-label="ES6 WeakMaps, Sets, and WeakSets in Depth on Pony Foo"><code class="md-code md-code-inline">WeakMap</code>, <code class="md-code md-code-inline">Set</code>, and <code class="md-code md-code-inline">WeakSet</code></a> for breakfast :)</p> </blockquote></div>
