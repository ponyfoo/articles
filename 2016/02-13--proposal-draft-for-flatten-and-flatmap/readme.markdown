<div></div>

<h1>Proposal Draft for <code class="md-code md-code-inline">.flatten</code> and <code class="md-code md-code-inline">.flatMap</code></h1>

<p><kbd>ecmascript</kbd> <kbd>flatten</kbd> <kbd>array</kbd> <kbd>proposal-draft</kbd></p>

<blockquote><p><code>Array</code> prototype may be getting <code>.flatten</code> and <code>.flatMap</code> methods may be coming to ECMAScript in a distant future. This article describes what the proposal holds in store.</p>
</blockquote>

<div><p><code class="md-code md-code-inline">Array</code> prototype may be getting <code class="md-code md-code-inline">.flatten</code> and <code class="md-code md-code-inline">.flatMap</code> methods may be coming to ECMAScript in a distant future. This article describes what the proposal holds in store.</p></div>

<div></div>

<div><p>A very early draft was <a href="http://bterlson.github.io/proposal-flatMap/" target="_blank" aria-label="Array.prototype.flatMap &amp; Array.prototype.flatten proposal">published last week</a> by the ECMAScript editor Brian Terlson. When I say very early I mean it&#x2019;s considered a <em>&#x201C;stage -1&#x201D;</em> proposal, meaning it&#x2019;s not even a formal proposal yet, just a very early draft.</p> <p>That being said, I&#x2019;m always excited about new <code class="md-code md-code-inline">Array.prototype</code> methods so I decided to write an article nonetheless. These kinds of methods were popularized in JavaScript by libraries like Underscore and then Lodash &#x2013; and some of them &#x2013; such as <code class="md-code md-code-inline">.includes</code>, have eventually started finding their way into the language.</p> <p>Shall we take a look?</p> <p><img src="https://i.imgur.com/gJJdfyS.jpg" alt="A car compactor"></p></div>

<div><h1 id="arrayprototypeflatten"><code class="md-code md-code-inline">Array.prototype.flatten</code></h1> <p>The <code class="md-code md-code-inline">.flatten</code> proposal will take an array and return a new array where the old array was flattened recursively. The following bits of code represent the <code class="md-code md-code-inline">Array.prototype.flatten</code> API.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-number">1</span>, <span class="md-code-number">2</span>, <span class="md-code-number">3</span>, <span class="md-code-number">4</span>].flatten() <span class="md-code-comment">// &lt;- [1, 2, 3, 4]</span>
[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, <span class="md-code-number">3</span>], <span class="md-code-number">4</span>].flatten() <span class="md-code-comment">// &lt;- [1, 2, 3, 4]</span>
[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, [<span class="md-code-number">3</span>]], <span class="md-code-number">4</span>].flatten() <span class="md-code-comment">// &lt;- [1, 2, 3, 4]</span>
</code></pre> <p>One could implement a polyfill for <code class="md-code md-code-inline">.flatten</code> thus far like below. I separated the implementation of <code class="md-code md-code-inline">flatten</code> from the polyfill so that you don&#x2019;t necessarily have to use it as a polyfill if you just want to use the method without changing <code class="md-code md-code-inline">Array.prototype</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.flatten = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> flatten(<span class="md-code-keyword">this</span>)
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">flatten</span> <span class="md-code-params">(list)</span> </span>{
  <span class="md-code-keyword">return</span> list.reduce((a, b) =&gt; (<span class="md-code-built_in">Array</span>.isArray(b) ? a.push(...flatten(b)) : a.push(b), a), [])
}
</code></pre> <p>Keep in mind that the code above might not be the most efficient approach to array flattening, but it accomplishes recursive array flattening in a few lines of code. Here&#x2019;s how it works.</p> <ul> <li>A consumer calls <code class="md-code md-code-inline">x.flatten()</code></li> <li>The <code class="md-code md-code-inline">x</code> list is reduced using <code class="md-code md-code-inline">.reduce</code> into a new array <code class="md-code md-code-inline">[]</code> named <code class="md-code md-code-inline">a</code></li> <li>Each item <code class="md-code md-code-inline">b</code> in <code class="md-code md-code-inline">x</code> is evaluated through <code class="md-code md-code-inline">Array.isArray</code></li> <li>Items that aren&#x2019;t an array are pushed to <code class="md-code md-code-inline">a</code></li> <li>Items that are an array are flattened into a new array <ul> <li>Those items are <a href="https://ponyfoo.com/articles/es6-spread-and-butter-in-depth" aria-label="ES6 Spread and Butter in Depth">spread over</a> a <code class="md-code md-code-inline">.push</code> call for <code class="md-code md-code-inline">a</code></li> </ul> </li> <li>This eventually results in a flat array</li> </ul> <p>The proposal also comes with an optional <code class="md-code md-code-inline">depth</code> parameter <em>&#x2013; that defaults to <code class="md-code md-code-inline">Infinity</code> &#x2013;</em> which can be used to determine how deep the flattening should go.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, [<span class="md-code-number">3</span>]], <span class="md-code-number">4</span>].flatten() <span class="md-code-comment">// &lt;- [1, 2, 3, 4]</span>
[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, [<span class="md-code-number">3</span>]], <span class="md-code-number">4</span>].flatten(<span class="md-code-number">2</span>) <span class="md-code-comment">// &lt;- [1, 2, 3, 4]</span>
[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, [<span class="md-code-number">3</span>]], <span class="md-code-number">4</span>].flatten(<span class="md-code-number">1</span>) <span class="md-code-comment">// &lt;- [1, 2, [3], 4]</span>
[<span class="md-code-number">1</span>, [<span class="md-code-number">2</span>, [<span class="md-code-number">3</span>]], <span class="md-code-number">4</span>].flatten(<span class="md-code-number">0</span>) <span class="md-code-comment">// &lt;- [1, [2, [3]], 4]</span>
</code></pre> <p>Adding the <code class="md-code md-code-inline">depth</code> option to our polyfill wouldn&#x2019;t be that hard, we pass it down to recursive <code class="md-code md-code-inline">flatten</code> calls and ensure that, <em>when the bottom is reached</em>, we stop flattening and recursion.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.flatten = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(<mark class="md-mark md-code-mark">depth=Infinity</mark>)</span> </span>{
  <span class="md-code-keyword">return</span> flatten(this<mark class="md-mark md-code-mark">, depth</mark>)
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">flatten</span> <span class="md-code-params">(list<mark class="md-mark md-code-mark">, depth</mark>)</span> </span>{
  <span class="md-code-keyword">if</span> (depth === <span class="md-code-number">0</span>) {
    <span class="md-code-keyword">return</span> list
  }
  <span class="md-code-keyword">return</span> list.reduce((accumulator, item) =&gt; {
    <span class="md-code-keyword">if</span> (<span class="md-code-built_in">Array</span>.isArray(item)) {
      accumulator.push(...flatten(item, <mark class="md-mark md-code-mark">depth - <span class="md-code-number">1</span></mark>))
    } <span class="md-code-keyword">else</span> {
      accumulator.push(item)
    }
    <span class="md-code-keyword">return</span> accumulator
  }, [])
}
</code></pre> <p>Alternatively <em>&#x2013; for Internet points &#x2013;</em> we could fit the whole of <code class="md-code md-code-inline">flatten</code> in a single expression.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">flatten</span> <span class="md-code-params">(list, depth)</span> </span>{
  <span class="md-code-keyword">return</span> depth === <span class="md-code-number">0</span> ? list : list.reduce((a, b) =&gt; (<span class="md-code-built_in">Array</span>.isArray(b) ?
    a.push(...flatten(b, depth - <span class="md-code-number">1</span>)) :
    a.push(b), a), [])
}
</code></pre> <p>Then there&#x2019;s <code class="md-code md-code-inline">.flatMap</code>.</p> <h1 id="arrayprototypeflatmap"><code class="md-code md-code-inline">Array.prototype.flatMap</code></h1> <p>This method is convenient because of how often use cases come up where it might be appropriate, and at the same time it provides a small boost in performance, as we&#x2019;ll note next.</p> <p>Taking into account the polyfill we created earlier for flattening through <code class="md-code md-code-inline">Array.prototype.flatten</code>, the <code class="md-code md-code-inline">.flatMap</code> method can be represented in code like below. Note how you can provide a mapping function <code class="md-code md-code-inline">fn</code> and its <code class="md-code md-code-inline">ctx</code> context as usual, but the flattening is fixed at a depth of <code class="md-code md-code-inline">1</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.flatMap = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(fn, ctx)</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>.map(fn, ctx).flatten(<span class="md-code-number">1</span>)
}
</code></pre> <p>Typically, the code shown above is how you would implement <code class="md-code md-code-inline">.flatMap</code> in user code, but the native <code class="md-code md-code-inline">.flatMap</code> <strong>trades a bit of readability for performance</strong>, by introducing the ability to map items directly in the internal flatten procedure, avoiding the two-pass that&#x2019;s necessary if we first <code class="md-code md-code-inline">.map</code> and then <code class="md-code md-code-inline">.flatten</code> an <code class="md-code md-code-inline">Array</code>.</p> <p>A possible example of using <code class="md-code md-code-inline">.flatMap</code> can be found below.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[{ x: <span class="md-code-number">1</span>, y: <span class="md-code-number">2</span> }, { x: <span class="md-code-number">3</span>, y: <span class="md-code-number">4</span> }, { x: <span class="md-code-number">5</span>, y: <span class="md-code-number">6</span> }].flatMap(c =&gt; [c.x, c.y])
<span class="md-code-comment">// &lt;- [1, 2, 3, 4, 5, 6]</span>
</code></pre> <p>The above is syntactic sugar for doing <code class="md-code md-code-inline">.map(c =&gt; [c.x, c.y]).flatten()</code> while providing a small performance boost by avoiding the aforementioned two-pass when first mapping and then flattening.</p> <p>Note that our previous polyfill <strong>doesn&#x2019;t cover the performance boost</strong>, let&#x2019;s fix that by changing our own internal <code class="md-code md-code-inline">flatten</code> function and adjust <code class="md-code md-code-inline">Array.prototype.flatMap</code> accordingly. We&#x2019;ve added a couple more parameters to <code class="md-code md-code-inline">flatten</code>, where we allow the item to be mapped into a different value right before flattening, and avoiding the extra loop over the array.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">Array</span>.prototype.flatMap = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(fn, ctx)</span> </span>{
  <span class="md-code-keyword">return</span> <mark class="md-mark md-code-mark">flatten(<span class="md-code-keyword">this</span>, <span class="md-code-number">1</span>, fn, ctx)</mark>
}
<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">flatten</span> <span class="md-code-params">(list, depth<mark class="md-mark md-code-mark">, mapperFn, mapperCtx</mark>)</span> </span>{
  <span class="md-code-keyword">if</span> (depth === <span class="md-code-number">0</span>) {
    <span class="md-code-keyword">return</span> list
  }
  <span class="md-code-keyword">return</span> list.reduce((accumulator, item<mark class="md-mark md-code-mark">, i</mark>) =&gt; {
    <span class="md-code-keyword">if</span> (mapperFn) {
      <mark class="md-mark md-code-mark">item = mapperFn.call(mapperCtx || list, item, i, list)</mark>
    }
    <span class="md-code-keyword">if</span> (<span class="md-code-built_in">Array</span>.isArray(item)) {
      accumulator.push(...flatten(item, depth - <span class="md-code-number">1</span>))
    } <span class="md-code-keyword">else</span> {
      accumulator.push(item)
    }
    <span class="md-code-keyword">return</span> accumulator
  }, [])
}
</code></pre> <blockquote> <p>Since the <code class="md-code md-code-inline">mapperFn</code> and <code class="md-code md-code-inline">mapperCtx</code> parameters of <code class="md-code md-code-inline">flatten</code> are entirely optional, we could still use this same internal <code class="md-code md-code-inline">flatten</code> function to polyfill both <code class="md-code md-code-inline">.flatten</code> and <code class="md-code md-code-inline">.flatMap</code>.</p> </blockquote></div>
