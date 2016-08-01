<div></div>

<h1>Fun with Native Arrays</h1>

<p><kbd>js</kbd> <kbd>js-native</kbd> <kbd>array</kbd></p>

<blockquote><p>In JavaScript, arrays can be created with the <code>Array</code> constructor, or using the <code>[]</code> convenience shortcut, which is also the preferred approach. Arrays inherit from the <code>&#x2026;</code></p></blockquote>

<div><p>In JavaScript, arrays can be created with the <code class="md-code md-code-inline">Array</code> constructor, or using the <code class="md-code md-code-inline">[]</code> convenience shortcut, which is also the preferred approach. Arrays inherit from the <code class="md-code md-code-inline">Object</code> prototype and they haven&#x2019;t a special value for <code class="md-code md-code-inline">typeof</code>, they return <code class="md-code md-code-inline">&apos;object&apos;</code> too. Using <code class="md-code md-code-inline">[] instanceof Array</code>, however, returns true. That being said, there are also <em>Array-like objects</em> which complicate matters, <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments" target="_blank">such as strings, or the <code class="md-code md-code-inline">arguments</code> object</a>. The <code class="md-code md-code-inline">arguments</code> object is not an instance of <code class="md-code md-code-inline">Array</code>, but it still has a <code class="md-code md-code-inline">length</code> property, and its values are indexed, so it can be looped like any Array.</p></div>

<div></div>

<div><p>In this article I&#x2019;ll go over a few of the methods on the <code class="md-code md-code-inline">Array</code> prototype, and explore the possibilities each of these methods unveil.</p> <ul> <li>Looping with <code class="md-code md-code-inline">.forEach</code></li> <li>Asserting with <code class="md-code md-code-inline">.some</code> and <code class="md-code md-code-inline">.every</code></li> <li>Subtleties in <code class="md-code md-code-inline">.join</code> and <code class="md-code md-code-inline">.concat</code></li> <li>Stacks and queues with <code class="md-code md-code-inline">.pop</code>, <code class="md-code md-code-inline">.push</code>, <code class="md-code md-code-inline">.shift</code>, and <code class="md-code md-code-inline">.unshift</code></li> <li>Model mapping with <code class="md-code md-code-inline">.map</code></li> <li>Querying with <code class="md-code md-code-inline">.filter</code></li> <li>Ordering with <code class="md-code md-code-inline">.sort</code></li> <li>Computing with <code class="md-code md-code-inline">.reduce</code>, <code class="md-code md-code-inline">.reduceRight</code></li> <li>Copying a <code class="md-code md-code-inline">.slice</code></li> <li>The power of <code class="md-code md-code-inline">.splice</code></li> <li>Lookups with <code class="md-code md-code-inline">.indexOf</code></li> <li>The <code class="md-code md-code-inline">in</code> operator</li> <li>Going in <code class="md-code md-code-inline">.reverse</code></li> </ul></div>

<div><p><img alt="console.png" class="" src="https://i.imgur.com/z0Hun2i.png"></p> <p>You can copy and paste any of the examples in your browser&#x2019;s console, I sure did!</p> <h3 id="looping-with-foreach">Looping with <code class="md-code md-code-inline">.forEach</code></h3> <p>This is one of the simplest methods in a native JavaScript Array. <a href="http://kangax.github.io/es5-compat-table/#Array.prototype.forEach" target="_blank" aria-label="ECMAScript 5 compatibility table">Unsurprisingly unsupported&#x2122;</a> in IE7 and IE8.</p> <p><code class="md-code md-code-inline">forEach</code> takes a callback which is invoked once for each element in the array, and gets passed three arguments.</p> <ul> <li><code class="md-code md-code-inline">value</code> containing the current array element</li> <li><code class="md-code md-code-inline">index</code> is the element&#x2019;s position in the array</li> <li><code class="md-code md-code-inline">array</code> is a reference to the array</li> </ul> <p>Furthermore, we could pass an optional second argument which will become the context (<code class="md-code md-code-inline">this</code>) for each function call.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-string">&apos;_&apos;</span>, <span class="md-code-string">&apos;t&apos;</span>, <span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-string">&apos;n&apos;</span>, <span class="md-code-string">&apos;i&apos;</span>, <span class="md-code-string">&apos;f&apos;</span>, <span class="md-code-string">&apos;]&apos;</span>].forEach(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value, index, array)</span> </span>{
	<span class="md-code-keyword">this</span>.push(<span class="md-code-built_in">String</span>.fromCharCode(value.charCodeAt() + index + <span class="md-code-number">2</span>))
}, out = [])

out.join(<span class="md-code-string">&apos;&apos;</span>)
<span class="md-code-comment">// &lt;- &apos;awesome&apos;</span>
</code></pre> <p>I cheated with <code class="md-code md-code-inline">.join</code> which we didn&#x2019;t cover <em>yet</em>, but we&#x2019;ll look at it soon. In this case, it joins together the different elements in the array, effectively doing something like <code class="md-code md-code-inline">out[0] + &apos;&apos; + out[1] + &apos;&apos; + out[2] + &apos;&apos; + out[n]</code>. We <strong>can&#x2019;t break <code class="md-code md-code-inline">forEach</code> loops</strong>, and throwing exceptions wouldn&#x2019;t be very sensible. Luckily, we have other options available to us in those cases where we might want to short-circuit a loop.</p> <h3 id="asserting-with-some-and-every">Asserting with <code class="md-code md-code-inline">.some</code> and <code class="md-code md-code-inline">.every</code></h3> <p>If you&#x2019;ve ever worked with .NET&#x2019;s enumerables, these methods are the <em>poorly named</em> cousins of <a href="http://msdn.microsoft.com/en-us/library/bb534972(v=vs.110).aspx" target="_blank" aria-label="IEnumerable.Any&lt;TSource&gt; in .NET 4.5"><code class="md-code md-code-inline">.Any(x =&gt; x.IsAwesome)</code></a> and <a href="http://msdn.microsoft.com/en-us/library/bb548541(v=vs.110).aspx" target="_blank" aria-label="IEnumerable.All&lt;TSource&gt; in .NET 4.5"><code class="md-code md-code-inline">.All(x =&gt; x.IsAwesome)</code></a>.</p> <p>These methods are similar to <code class="md-code md-code-inline">.forEach</code> in that they also take a callback with <code class="md-code md-code-inline">value</code>, <code class="md-code md-code-inline">index</code>, and <code class="md-code md-code-inline">array</code>, which can be context-bound passing a second argument. The MDN docs describe <code class="md-code md-code-inline">.some</code>:</p> <blockquote> <p><code class="md-code md-code-inline">some</code> executes the callback function once for each element present in the array until it finds one where <code class="md-code md-code-inline">callback</code> returns a <code class="md-code md-code-inline">true</code> value. If such an element is found, <code class="md-code md-code-inline">some</code> immediately returns <code class="md-code md-code-inline">true</code>. Otherwise, <code class="md-code md-code-inline">some</code> returns <code class="md-code md-code-inline">false</code>. <code class="md-code md-code-inline">callback</code> is invoked only for indexes of the array which have assigned values; it is not invoked for indexes which have been deleted or which have never been assigned values.</p> </blockquote> <pre class="md-code-block"><code class="md-code md-lang-javascript">max = -<span class="md-code-literal">Infinity</span>
satisfied = [<span class="md-code-number">10</span>, <span class="md-code-number">12</span>, <span class="md-code-number">10</span>, <span class="md-code-number">8</span>, <span class="md-code-number">5</span>, <span class="md-code-number">23</span>].some(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value, index, array)</span> </span>{
	<span class="md-code-keyword">if</span> (value &gt; max) max = value
	<span class="md-code-keyword">return</span> value &lt; <span class="md-code-number">10</span>
})

<span class="md-code-built_in">console</span>.log(max)
<span class="md-code-comment">// &lt;- 12</span>

satisfied
<span class="md-code-comment">// &lt;- true</span>
</code></pre> <p>Note that the function stopped looping after it hit the first item which satisfied the callback&#x2019;s condition <code class="md-code md-code-inline">value &lt; 10</code>. <code class="md-code md-code-inline">.every</code> works in the same way, but short-circuits happen when your callback returns <code class="md-code md-code-inline">false</code>, rather than <code class="md-code md-code-inline">true</code>.</p> <h3 id="subtleties-in-join-and-concat">Subtleties in <code class="md-code md-code-inline">.join</code> and <code class="md-code md-code-inline">.concat</code></h3> <p>The <code class="md-code md-code-inline">.join</code> method is often confused with <code class="md-code md-code-inline">.concat</code>. <code class="md-code md-code-inline">.join(separator)</code> creates a string, resulting of taking every element in the array and separating them by <code class="md-code md-code-inline">separator</code>. If no <code class="md-code md-code-inline">separator</code> is provided, it&#x2019;ll default to a comma <code class="md-code md-code-inline">&apos;,&apos;</code>. <code class="md-code md-code-inline">.concat</code> works by creating new arrays which are shallow copies of the source arrays.</p> <ul> <li><code class="md-code md-code-inline">.concat</code> has the signature: <code class="md-code md-code-inline">array.concat(val, val2, val3, valn)</code></li> <li><code class="md-code md-code-inline">.concat</code> returns a new array</li> <li><code class="md-code md-code-inline">array.concat()</code> with no arguments returns a shallow copy of the array</li> </ul> <p>Shallow copy means that the copy will hold the same object references as the source array, which is generally a good thing. For example:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> a = { foo: <span class="md-code-string">&apos;bar&apos;</span> }
<span class="md-code-keyword">var</span> b = [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>, a]
<span class="md-code-keyword">var</span> c = b.concat()

<span class="md-code-built_in">console</span>.log(b === c)
<span class="md-code-comment">// &lt;- false</span>

b[<span class="md-code-number">3</span>] === a &amp;&amp; c[<span class="md-code-number">3</span>] === a
<span class="md-code-comment">// &lt;- true</span>
</code></pre> <h3 id="stacks-and-queues-with-pop-push-shift-and-unshift">Stacks and queues with <code class="md-code md-code-inline">.pop</code>, <code class="md-code md-code-inline">.push</code>, <code class="md-code md-code-inline">.shift</code>, and <code class="md-code md-code-inline">.unshift</code></h3> <p>Nowadays, everyone knows that <em>adding elements to the end of an array</em> is done using <code class="md-code md-code-inline">.push</code>. Did you know that you can push many elements at once using <code class="md-code md-code-inline">[].push(&apos;a&apos;, &apos;b&apos;, &apos;c&apos;, &apos;d&apos;, &apos;z&apos;)</code>?</p> <p>The <code class="md-code md-code-inline">.pop</code> method is the counterpart of the most common use for <code class="md-code md-code-inline">.push</code>. It&#x2019;ll return the last element in the array, and remove it from the array at the same time. If the array is empty, <a href="http://stackoverflow.com/questions/7452341/what-does-void-0-mean" target="_blank" aria-label="void operator - MDN"><code class="md-code md-code-inline">void 0</code> (<code class="md-code md-code-inline">undefined</code>)</a> is returned. Using <code class="md-code md-code-inline">.push</code> and <code class="md-code md-code-inline">.pop</code> we could easily create a <a href="http://en.wikipedia.org/wiki/LIFO_(computing)" target="_blank" aria-label="LIFO on Wikipedia">LIFO (last in first out)</a> stack.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">Stack</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">this</span>._stack = []
}

Stack.prototype.next = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>._stack.pop()
}

Stack.prototype.add = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>._stack.push.apply(<span class="md-code-keyword">this</span>._stack, <span class="md-code-built_in">arguments</span>)
}

stack = <span class="md-code-keyword">new</span> Stack()
stack.add(<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>)

stack.next()
<span class="md-code-comment">// &lt;- 3</span>
</code></pre> <p>Inversely, we could create a <a href="http://en.wikipedia.org/wiki/FIFO" target="_blank" aria-label="FIFO on Wikipedia">FIFO (first in first out)</a> queue using <code class="md-code md-code-inline">.unshift</code> and <code class="md-code md-code-inline">.shift</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">Queue</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">this</span>._queue = []
}

Queue.prototype.next = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>._queue.shift()
}

Queue.prototype.add = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>._queue.unshift.apply(<span class="md-code-keyword">this</span>._queue, <span class="md-code-built_in">arguments</span>)
}

queue = <span class="md-code-keyword">new</span> Queue()
queue.add(<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>)

queue.next()
<span class="md-code-comment">// &lt;- 1</span>
</code></pre> <p>Using <code class="md-code md-code-inline">.shift</code> (or <code class="md-code md-code-inline">.pop</code>) is an easy way to loop through a set of array elements, while draining the array in the process.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">list = [<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>,<span class="md-code-number">4</span>,<span class="md-code-number">5</span>,<span class="md-code-number">6</span>,<span class="md-code-number">7</span>,<span class="md-code-number">8</span>,<span class="md-code-number">9</span>,<span class="md-code-number">10</span>]

<span class="md-code-keyword">while</span> (item = list.shift()) {
	<span class="md-code-built_in">console</span>.log(item)
}

list
<span class="md-code-comment">// &lt;- []</span>
</code></pre> <h3 id="model-mapping-with-map">Model mapping with <code class="md-code md-code-inline">.map</code></h3> <blockquote> <p><code class="md-code md-code-inline">map</code> calls a provided callback function once for each element in an array, in order, and constructs a new array from the results. <code class="md-code md-code-inline">callback</code> is invoked only for indexes of the array which have assigned values; it is not invoked for indexes which have been deleted or which have never been assigned values.</p> </blockquote> <p>The <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map" target="_blank" aria-label="Array.prototype.map - MDN"><code class="md-code md-code-inline">Array.prototype.map</code></a> method has the same signature we&#x2019;ve seen in <code class="md-code md-code-inline">.forEach</code>, <code class="md-code md-code-inline">.some</code>, and <code class="md-code md-code-inline">.every</code>: <code class="md-code md-code-inline">.map(fn(value, index, array), thisArgument)</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">values = [<span class="md-code-keyword">void</span> <span class="md-code-number">0</span>, <span class="md-code-literal">null</span>, <span class="md-code-literal">false</span>, <span class="md-code-string">&apos;&apos;</span>]
values[<span class="md-code-number">7</span>] = <span class="md-code-keyword">void</span> <span class="md-code-number">0</span>
result = values.map(<span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(value, index, array)</span></span>{
    <span class="md-code-built_in">console</span>.log(value)
    <span class="md-code-keyword">return</span> value
})

<span class="md-code-comment">// &lt;- [undefined, null, false, &apos;&apos;, undefined &#xD7; 3, undefined]</span>
</code></pre> <p>The <code class="md-code md-code-inline">undefined &#xD7; 3</code> values explain that while <code class="md-code md-code-inline">.map</code> won&#x2019;t run for deleted or unassigned array elements, they&#x2019;ll be still included in the resulting array. Mapping is very useful for casting or transforming arrays.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-comment">// casting</span>
[<span class="md-code-number">1</span>, <span class="md-code-string">&apos;2&apos;</span>, <span class="md-code-string">&apos;30&apos;</span>, <span class="md-code-string">&apos;9&apos;</span>].map(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-built_in">parseInt</span>(value, <span class="md-code-number">10</span>)
})
<span class="md-code-comment">// 1, 2, 30, 9</span>

[<span class="md-code-number">97</span>, <span class="md-code-number">119</span>, <span class="md-code-number">101</span>, <span class="md-code-number">115</span>, <span class="md-code-number">111</span>, <span class="md-code-number">109</span>, <span class="md-code-number">101</span>].map(<span class="md-code-built_in">String</span>.fromCharCode).join(<span class="md-code-string">&apos;&apos;</span>)
<span class="md-code-comment">// &lt;- &apos;awesome&apos;</span>

<span class="md-code-comment">// a commonly used pattern is mapping to new objects</span>
items.map(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(item)</span> </span>{
	<span class="md-code-keyword">return</span> {
		id: item.id,
		name: computeName(item)
	}
})
</code></pre> <h3 id="querying-with-filter">Querying with <code class="md-code md-code-inline">.filter</code></h3> <blockquote> <p><code class="md-code md-code-inline">filter</code> calls a provided callback function once for each element in an array, and constructs a new array of all the values for which <code class="md-code md-code-inline">callback</code> returns a <code class="md-code md-code-inline">true</code> value. <code class="md-code md-code-inline">callback</code> is invoked only for indexes of the array which have assigned values; it is not invoked for indexes which have been deleted or which have never been assigned values. Array elements which do not pass the <code class="md-code md-code-inline">callback</code> test are simply skipped, and <strong>are not included</strong> in the new array.</p> </blockquote> <p>Same as usual: <code class="md-code md-code-inline">.filter(fn(value, index, array), thisArgument)</code>. Think of it as the <code class="md-code md-code-inline">.Where(x =&gt; x.IsAwesome)</code> LINQ expression (if you&#x2019;re into C#), or the <code class="md-code md-code-inline">WHERE</code> SQL clause. Considering <code class="md-code md-code-inline">.filter</code> only returns elements which pass the <code class="md-code md-code-inline">callback</code> test with a truthy value, there are some interesting use cases.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-keyword">void</span> <span class="md-code-number">0</span>, <span class="md-code-literal">null</span>, <span class="md-code-literal">false</span>, <span class="md-code-string">&apos;&apos;</span>, <span class="md-code-number">1</span>].filter(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
	<span class="md-code-keyword">return</span> value
})
<span class="md-code-comment">// &lt;- [1]</span>

[<span class="md-code-keyword">void</span> <span class="md-code-number">0</span>, <span class="md-code-literal">null</span>, <span class="md-code-literal">false</span>, <span class="md-code-string">&apos;&apos;</span>, <span class="md-code-number">1</span>].filter(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
	<span class="md-code-keyword">return</span> !value
})
<span class="md-code-comment">// &lt;- [void 0, null, false, &apos;&apos;]</span>
</code></pre> <h3 id="ordering-with-sort-comparefunction">Ordering with <code class="md-code md-code-inline">.sort(compareFunction)</code></h3> <blockquote> <p>If <code class="md-code md-code-inline">compareFunction</code> is not supplied, elements are sorted by converting them to strings and comparing strings in lexicographic (&#x201C;dictionary&#x201D; or &#x201C;telephone book,&#x201D; not numerical) order. For example, &#x201C;80&#x201D; comes before &#x201C;9&#x201D; in lexicographic order, but in a numeric sort 9 comes before 80.</p> </blockquote> <p>Like most sorting functions, <code class="md-code md-code-inline">Array.prototype.sort(fn(a,b))</code> takes a callback which tests two elements, and should produce one of three return values:</p> <ul> <li>return value <code class="md-code md-code-inline">&lt; 0</code> if <code class="md-code md-code-inline">a</code> comes before <code class="md-code md-code-inline">b</code></li> <li>return value <code class="md-code md-code-inline">=== 0</code> if both <code class="md-code md-code-inline">a</code> and <code class="md-code md-code-inline">b</code> are considered equivalent</li> <li>return value <code class="md-code md-code-inline">&gt; 0</code> if <code class="md-code md-code-inline">a</code> comes after <code class="md-code md-code-inline">b</code></li> </ul> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-number">9</span>,<span class="md-code-number">80</span>,<span class="md-code-number">3</span>,<span class="md-code-number">10</span>,<span class="md-code-number">5</span>,<span class="md-code-number">6</span>].sort()
<span class="md-code-comment">// &lt;- [10, 3, 5, 6, 80, 9]</span>

[<span class="md-code-number">9</span>,<span class="md-code-number">80</span>,<span class="md-code-number">3</span>,<span class="md-code-number">10</span>,<span class="md-code-number">5</span>,<span class="md-code-number">6</span>].sort(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(a, b)</span> </span>{
	<span class="md-code-keyword">return</span> a - b
})
<span class="md-code-comment">// &lt;- [3, 5, 6, 9, 10, 80]</span>
</code></pre> <h3 id="computing-with-reduce-reduceright">Computing with <code class="md-code md-code-inline">.reduce</code>, <code class="md-code md-code-inline">.reduceRight</code></h3> <p>Reduce functions are, at first, hard to wrap our heads around. These functions loop through the array, from left-to-right <em>(<code class="md-code md-code-inline">.reduce</code>)</em> or right-to-left <em>(<code class="md-code md-code-inline">.reduceRight</code>)</em>, each invocation receives the partial result so far, and the operation results in a single aggregated return value.</p> <p>Both methods have the following signature: <code class="md-code md-code-inline">.reduce(callback(previousValue, currentValue, index, array), initialValue)</code>.</p> <p>The <code class="md-code md-code-inline">previousValue</code> will be the value returned in the last callback invocation, or <code class="md-code md-code-inline">initialValue</code> the first time around. <code class="md-code md-code-inline">currentValue</code> contains the current element, while <code class="md-code md-code-inline">index</code> indicates the array position for the element. <code class="md-code md-code-inline">array</code> is simply a reference to the array <code class="md-code md-code-inline">.reduce</code> was called on.</p> <p>One of the typical use cases for <code class="md-code md-code-inline">.reduce</code> is the sum function.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.sum = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
	<span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>.reduce(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(partial, value)</span> </span>{
		<span class="md-code-keyword">return</span> partial + value
	}, <span class="md-code-number">0</span>)
};

[<span class="md-code-number">3</span>,<span class="md-code-number">4</span>,<span class="md-code-number">5</span>,<span class="md-code-number">6</span>,<span class="md-code-number">10</span>].sum()
<span class="md-code-comment">// &lt;- 28</span>
</code></pre> <p>Say we wanted to join a few strings together. We could use <code class="md-code md-code-inline">.join</code> to that purpose. In the case of objects, though, <code class="md-code md-code-inline">.join</code> wouldn&#x2019;t work as we expected, unless the objects had a reasonable <code class="md-code md-code-inline">valueOf</code> or <code class="md-code md-code-inline">toString</code> representation. However, we might use <code class="md-code md-code-inline">.reduce</code> as a string builder for those objects.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">concat</span> <span class="md-code-params">(input)</span> </span>{
	<span class="md-code-keyword">return</span> input.reduce(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(partial, value)</span> </span>{
		<span class="md-code-keyword">if</span> (partial) {
			partial += <span class="md-code-string">&apos;, &apos;</span>
		}
		<span class="md-code-keyword">return</span> partial + value.name
	}, <span class="md-code-string">&apos;&apos;</span>)
}

concat([
	{ name: <span class="md-code-string">&apos;George&apos;</span> },
	{ name: <span class="md-code-string">&apos;Sam&apos;</span> },
	{ name: <span class="md-code-string">&apos;Pear&apos;</span> }
])
<span class="md-code-comment">// &lt;- &apos;George, Sam, Pear&apos;</span>
</code></pre> <h3 id="copying-a-slice">Copying a <code class="md-code md-code-inline">.slice</code></h3> <p>Similarly to <code class="md-code md-code-inline">.concat</code>, calls to <code class="md-code md-code-inline">.slice</code> without any arguments produce a shallow copy of the source array. Slice takes two arguments, a <code class="md-code md-code-inline">begin</code> and an <code class="md-code md-code-inline">end</code> position. <code class="md-code md-code-inline">Array.prototype.slice</code> can be used to convert array-like objects into real arrays.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.slice.call({ <span class="md-code-number">0</span>: <span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-number">1</span>: <span class="md-code-string">&apos;b&apos;</span>, length: <span class="md-code-number">2</span> })
<span class="md-code-comment">// &lt;- [&apos;a&apos;, &apos;b&apos;]</span>
</code></pre> <p>This won&#x2019;t work with <code class="md-code md-code-inline">.concat</code>, because it&#x2019;ll wrap the array-like object in a real array, instead.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.concat.call({ <span class="md-code-number">0</span>: <span class="md-code-string">&apos;a&apos;</span>, <span class="md-code-number">1</span>: <span class="md-code-string">&apos;b&apos;</span>, length: <span class="md-code-number">2</span> })
<span class="md-code-comment">// &lt;- [{ 0: &apos;a&apos;, 1: &apos;b&apos;, length: 2 }]</span>
</code></pre> <p>Other than that, another common use for <code class="md-code md-code-inline">.slice</code> is <em>removing the first few elements</em> from a list of arguments (an array-like object, which we could cast to a real array).</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">format</span> <span class="md-code-params">(text, bold)</span> </span>{
	<span class="md-code-keyword">if</span> (bold) {
		text = <span class="md-code-string">&apos;&lt;b&gt;&apos;</span> + text + <span class="md-code-string">&apos;&lt;/b&gt;&apos;</span>
	}
	<span class="md-code-keyword">var</span> values = <span class="md-code-built_in">Array</span>.prototype.slice.call(<span class="md-code-built_in">arguments</span>, <span class="md-code-number">2</span>)

	values.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
		text = text.replace(<span class="md-code-string">&apos;%s&apos;</span>, value)
	})

	<span class="md-code-keyword">return</span> text
}

format(<span class="md-code-string">&apos;some%sthing%s %s&apos;</span>, <span class="md-code-literal">true</span>, <span class="md-code-string">&apos;some&apos;</span>, <span class="md-code-string">&apos;other&apos;</span>, <span class="md-code-string">&apos;things&apos;</span>)
<span class="md-code-comment">// &lt;- &lt;b&gt;somesomethingother things&lt;/b&gt;</span>
</code></pre> <h3 id="the-power-of-splice">The power of <code class="md-code md-code-inline">.splice</code></h3> <p><code class="md-code md-code-inline">.splice</code> is one of my favorite native array functions. It allows you to remove elements, insert new ones, and to do both in the same position, using just one function call. Note that this function alters the source array, unlike <code class="md-code md-code-inline">.concat</code> or <code class="md-code md-code-inline">.slice</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> source = [<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">9</span>,<span class="md-code-number">10</span>,<span class="md-code-number">11</span>,<span class="md-code-number">12</span>,<span class="md-code-number">13</span>]
<span class="md-code-keyword">var</span> spliced = source.splice(<span class="md-code-number">3</span>, <span class="md-code-number">4</span>, <span class="md-code-number">4</span>, <span class="md-code-number">5</span>, <span class="md-code-number">6</span>, <span class="md-code-number">7</span>)

<span class="md-code-built_in">console</span>.log(source)
<span class="md-code-comment">// &lt;- [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 ,13]</span>

spliced
<span class="md-code-comment">// &lt;- [8, 8, 8, 8]</span>
</code></pre> <p>As you might&#x2019;ve noted, it also returns the removed elements. This might come in handy if you want to loop a section of the array and then forget about it.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> source = [<span class="md-code-number">1</span>,<span class="md-code-number">2</span>,<span class="md-code-number">3</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">8</span>,<span class="md-code-number">9</span>,<span class="md-code-number">10</span>,<span class="md-code-number">11</span>,<span class="md-code-number">12</span>,<span class="md-code-number">13</span>]
<span class="md-code-keyword">var</span> spliced = source.splice(<span class="md-code-number">9</span>)

spliced.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(value)</span> </span>{
	<span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;removed&apos;</span>, value)
})
<span class="md-code-comment">// &lt;- removed 10</span>
<span class="md-code-comment">// &lt;- removed 11</span>
<span class="md-code-comment">// &lt;- removed 12</span>
<span class="md-code-comment">// &lt;- removed 13</span>

<span class="md-code-built_in">console</span>.log(source)
<span class="md-code-comment">// &lt;- [1, 2, 3, 8, 8, 8, 8, 8, 9]</span>
</code></pre> <h3 id="lookups-with-indexof">Lookups with <code class="md-code md-code-inline">.indexOf</code></h3> <p>With <code class="md-code md-code-inline">.indexOf</code>, we can look up array element positions. If it can&#x2019;t find a match, <code class="md-code md-code-inline">-1</code> is returned. A pattern I find myself using a lot, is when I have comparisons such as <code class="md-code md-code-inline">a === &apos;a&apos; || a === &apos;b&apos; || a === &apos;c&apos;</code>, or even with just two comparsions. You could just use <code class="md-code md-code-inline">.indexOf</code>, like so: <code class="md-code md-code-inline">[&apos;a&apos;, &apos;b&apos;, &apos;c&apos;].indexOf(a) !== -1</code>.</p> <p>Note that objects will be found only if the same reference is provided. A second argument can provide the start index at which to begin searching.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> a = { foo: <span class="md-code-string">&apos;bar&apos;</span> }
<span class="md-code-keyword">var</span> b = [a, <span class="md-code-number">2</span>]

<span class="md-code-built_in">console</span>.log(b.indexOf(<span class="md-code-number">1</span>))
<span class="md-code-comment">// &lt;- -1</span>

<span class="md-code-built_in">console</span>.log(b.indexOf({ foo: <span class="md-code-string">&apos;bar&apos;</span> }))
<span class="md-code-comment">// &lt;- -1</span>

<span class="md-code-built_in">console</span>.log(b.indexOf(a))
<span class="md-code-comment">// &lt;- 0</span>

<span class="md-code-built_in">console</span>.log(b.indexOf(a, <span class="md-code-number">1</span>))
<span class="md-code-comment">// &lt;- -1</span>

b.indexOf(<span class="md-code-number">2</span>, <span class="md-code-number">1</span>)
<span class="md-code-comment">// &lt;- 1</span>
</code></pre> <p>If you want to go in the reverse direction, <code class="md-code md-code-inline">.lastIndexOf</code> will do the trick.</p> <h3 id="the-in-operator">The <code class="md-code md-code-inline">in</code> operator</h3> <p>A common rookie mistake during interviews is to confuse <code class="md-code md-code-inline">.indexOf</code> with the <code class="md-code md-code-inline">in</code> operator, and hand-scribbling things such as:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> a = [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">5</span>]

<span class="md-code-number">1</span> <span class="md-code-keyword">in</span> a
<span class="md-code-comment">// &lt;- true, but because of the 2!</span>

<span class="md-code-number">5</span> <span class="md-code-keyword">in</span> a
<span class="md-code-comment">// &lt;- false</span>
</code></pre> <p>The problem here was that the <code class="md-code md-code-inline">in</code> operator checks the object key for a value, rather than searching for values. This is, of course, much faster than using <code class="md-code md-code-inline">.indexOf</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> a = [<span class="md-code-number">3</span>, <span class="md-code-number">7</span>, <span class="md-code-number">6</span>]

<span class="md-code-number">1</span> <span class="md-code-keyword">in</span> a === !!a[<span class="md-code-number">1</span>]
<span class="md-code-comment">// &lt;- true</span>
</code></pre> <p>The <code class="md-code md-code-inline">in</code> operator is similar to casting the value at the provided key to a boolean value. The <code class="md-code md-code-inline">!!</code> expression is used by some developers to negate a value, and then negate it again. <em>Effectively casting to boolean</em> any truthy value to <code class="md-code md-code-inline">true</code>, and any falsy value to <code class="md-code md-code-inline">false</code>.</p> <h3 id="going-in-reverse">Going in <code class="md-code md-code-inline">.reverse</code></h3> <p>This method will take the elements in an array and reverse them in place.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> a = [<span class="md-code-number">1</span>, <span class="md-code-number">1</span>, <span class="md-code-number">7</span>, <span class="md-code-number">8</span>]

a.reverse()
<span class="md-code-comment">// [8, 7, 1, 1]</span>
</code></pre> <p>Rather than a copy, the array itself is modified. In a future article we&#x2019;ll expand on these concepts to see how we could create an <code class="md-code md-code-inline">_</code>-like library, such as <a href="http://underscorejs.org/" target="_blank" aria-label="Underscore.js utility belt">Underscore</a> or <a href="http://lodash.com/" target="_blank" aria-label="Lo-Dash utility library">Lo-Dash</a>.</p></div>
