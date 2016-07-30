<div><blockquote>
  <h1>ES6 Generators in Depth</h1>
  <div><p>This is <a href="http://localhost:3000/articles/tagged/es6-in-depth">ES6 in Depth</a>, the longest-running article series in the history of Pony Foo! Trapped in the ES5 bubble? Welcome! Let me get you started with <a href="http://localhost:3000/articles/es6-destructuring-in-depth">destructuring</a>, <a href="http://localhost:3000/articles/es6-template-strings-in-depth">&#x2026;</a></p></div>
</blockquote></div>

<div><p>This is <a href="http://localhost:3000/articles/tagged/es6-in-depth">ES6 in Depth</a>, the longest-running article series in the history of Pony Foo! Trapped in the ES5 bubble? Welcome! Let me get you started with <a href="http://localhost:3000/articles/es6-destructuring-in-depth">destructuring</a>, <a href="http://localhost:3000/articles/es6-template-strings-in-depth">template literals</a>, <a href="http://localhost:3000/articles/es6-arrow-functions-in-depth">arrow functions</a>, the <a href="http://localhost:3000/articles/es6-spread-and-butter-in-depth">spread operator and rest parameters</a>, improvements coming to <a href="http://localhost:3000/articles/es6-object-literal-features-in-depth">object literals</a>, the new <a href="http://localhost:3000/articles/es6-classes-in-depth"><em>classes</em></a> sugar on top of prototypes, <a href="http://localhost:3000/articles/es6-let-const-and-temporal-dead-zone-in-depth"><code class="md-code md-code-inline">let</code>, <code class="md-code md-code-inline">const</code>, and the <em>&#x201C;Temporal Dead Zone&#x201D;</em></a>, and <a href="http://localhost:3000/articles/es6-iterators-in-depth">Iterators</a>.</p></div>

<div></div>

<div><blockquote> <p>Like I did in previous articles on the series, I would love to point out that you should probably <a href="http://localhost:3000/articles/universal-react-babel#setting-up-babel">set up Babel</a> and follow along the examples with either a REPL or the <code class="md-code md-code-inline">babel-node</code> CLI and a file. That&#x2019;ll make it so much easier for you to <strong>internalize the concepts</strong> discussed in the series. If you aren&#x2019;t the <em>&#x201C;install things on my computer&#x201D;</em> kind of human, you might prefer to hop on <a href="http://codepen.io/" target="_blank">CodePen</a> and then click on the gear icon for JavaScript &#x2013; <em>they have a Babel preprocessor which makes trying out ES6 a breeze.</em> Another alternative that&#x2019;s also quite useful is to use Babel&#x2019;s <a href="http://babeljs.io/repl/" target="_blank">online REPL</a> <em>&#x2013; it&#x2019;ll show you compiled ES5 code to the right of your ES6 code for quick comparison.</em></p> </blockquote> <p>Before getting into it, let me <a href="https://www.patreon.com/bevacqua" target="_blank"><em>shamelessly ask for your support</em></a> if you&#x2019;re enjoying my <a href="http://localhost:3000/articles/tagged/es6-in-depth">ES6 in Depth</a> series. Your contributions will go towards helping me keep up with the schedule, server bills, keeping me fed, and maintaining <strong>Pony Foo</strong> as a veritable source of JavaScript goodies.</p> <p>Thanks for listening to that, and let&#x2019;s go into generators now! If you haven&#x2019;t yet, you should read yesterday&#x2019;s article on <a href="http://localhost:3000/articles/es6-iterators-in-depth">iterators</a>, as this article pretty much assumes that you&#x2019;ve read it.</p></div>

<div><h2 id="generator-functions-and-generator-objects">Generator Functions and Generator Objects</h2> <p>Generators are a new feature in ES6. You declare a <em>generator function</em> which returns generator objects <code class="md-code md-code-inline">g</code> that can then be iterated using any of <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from" target="_blank" aria-label="Array.from() on MDN"><code class="md-code md-code-inline">Array.from(g)</code></a>, <a href="http://localhost:3000/articles/es6-spread-and-butter-in-depth" aria-label="ES6 Spread and Butter in Depth on Pony Foo"><code class="md-code md-code-inline">[...g]</code></a>, or <a href="http://localhost:3000/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo"><code class="md-code md-code-inline">for value of g</code></a> loops. Generator functions allow you to declare a special kind of <em>iterator</em>. These iterators can suspend execution while retaining their context. We already examined iterators in <a href="http://localhost:3000/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo">the previous article</a> and how their <code class="md-code md-code-inline">.next()</code> method is called once at a time to pull values from a sequence.</p> <p>Here is an example generator function. Note the <code class="md-code md-code-inline">*</code> after <code class="md-code md-code-inline">function</code>. That&#x2019;s not a typo, that&#x2019;s how you mark a generator function as a <em>generator</em>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">generator</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;f&apos;</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;o&apos;</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;o&apos;</span>
}
</code></pre> <p>Generator objects conform to both the <em>iterable</em> protocol and the <em>iterator</em> protocol. This means&#x2026;</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> g = <mark class="md-mark md-code-mark">generator()</mark>
<span class="md-code-comment">// a generator object g is built using the generator function</span>
<span class="md-code-keyword">typeof</span> g[Symbol.iterator] === <span class="md-code-string">&apos;function&apos;</span>
<span class="md-code-comment">// it&apos;s an iterable because it has an @@iterator</span>
<span class="md-code-keyword">typeof</span> g.next === <span class="md-code-string">&apos;function&apos;</span>
<span class="md-code-comment">// it&apos;s also an iterator because it has a .next method</span>
g[Symbol.iterator]() === g
<span class="md-code-comment">// the iterator for a generator object is the generator object itself</span>
<span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">[...g]</mark>)
<span class="md-code-comment">// &lt;- [&apos;f&apos;, &apos;o&apos;, &apos;o&apos;]</span>
<span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">Array.from(g)</mark>)
<span class="md-code-comment">// &lt;- [&apos;f&apos;, &apos;o&apos;, &apos;o&apos;]</span>
</code></pre> <p><sub><em>(This article is starting to sound an awful lot like a Math course&#x2026;)</em></sub></p> <p>When you create a generator object <em>(I&#x2019;ll just call them &#x201C;generator&#x201D; from here on out)</em>, you&#x2019;ll get an <em>iterator</em> that uses the generator to produce its <em>sequence</em>. Whenever a <code class="md-code md-code-inline">yield</code> expression is reached, that value is emitted by the iterator and <strong>function execution is suspended</strong>.</p> <p>Let&#x2019;s use a different example, this time with some other statements mixed in between <code class="md-code md-code-inline">yield</code> expressions. This is a simple generator but it behaves in an interesting enough way for our purposes here.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">generator</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;p&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;o&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;n&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;y&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;f&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;o&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;o&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;!&apos;</span>)
}
</code></pre> <p>If we use a <code class="md-code md-code-inline">for..of</code> loop, this will print <code class="md-code md-code-inline">ponyfoo!</code> one character at a time, as expected.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = generator()
<span class="md-code-keyword">for</span> (<mark class="md-mark md-code-mark">let pony of foo</mark>) {
  <span class="md-code-built_in">console</span>.log(pony)
  <span class="md-code-comment">// &lt;- &apos;p&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;o&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;n&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;y&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;f&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;o&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;o&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;!&apos;</span>
}
</code></pre> <p>What about using the spread <code class="md-code md-code-inline">[...foo]</code> syntax? Things turn out a little different here. This might be a little unexpected, but that&#x2019;s how generators work, everything that&#x2019;s not yielded ends up becoming <strong>a side effect</strong>. As the sequence is being constructed, the <code class="md-code md-code-inline">console.log</code> statements in between <code class="md-code md-code-inline">yield</code> calls are executed, and they print characters to the console before <code class="md-code md-code-inline">foo</code> is spread over an array. The previous example worked because we were printing characters as soon as they were pulled from the sequence, instead of waiting to construct a range for the entire sequence first.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = generator()
<span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">[...foo]</mark>)
<span class="md-code-comment">// &lt;- &apos;o&apos;</span>
<span class="md-code-comment">// &lt;- &apos;y&apos;</span>
<span class="md-code-comment">// &lt;- &apos;o&apos;</span>
<span class="md-code-comment">// &lt;- &apos;!&apos;</span>
<span class="md-code-comment">// &lt;- [&apos;p&apos;, &apos;n&apos;, &apos;f&apos;, &apos;o&apos;]</span>
</code></pre> <p>A neat aspect of generator functions is that you can also use <code class="md-code md-code-inline">yield*</code> to delegate to another generator function. Want a very contrived way to split <code class="md-code md-code-inline">&apos;ponyfoo&apos;</code> into individual characters? Since strings in ES6 adhere to the <em>iterable</em> protocol, you could do the following.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">generator</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span>* <span class="md-code-string">&apos;ponyfoo&apos;</span>
}
<span class="md-code-built_in">console</span>.log([...generator()])
<span class="md-code-comment">// &lt;- [&apos;p&apos;, &apos;o&apos;, &apos;n&apos;, &apos;y&apos;, &apos;f&apos;, &apos;o&apos;, &apos;o&apos;]</span>
</code></pre> <p>Of course, in the real world you could just do <code class="md-code md-code-inline">[...&apos;ponyfoo&apos;]</code>, since spread supports iterables just fine. Just like you could <code class="md-code md-code-inline">yield*</code> a string, you can <code class="md-code md-code-inline">yield*</code> anything that adheres to the iterable protocol. That includes other generators, arrays, and come ES6 &#x2013; <em>just about anything.</em></p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> foo = {
  [Symbol.iterator]: () =&gt; ({
    items: <mark class="md-mark md-code-mark">[<span class="md-code-string">&apos;p&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>, <span class="md-code-string">&apos;n&apos;</span>, <span class="md-code-string">&apos;y&apos;</span>, <span class="md-code-string">&apos;f&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>, <span class="md-code-string">&apos;o&apos;</span>]</mark>,
    next: <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">next</span> <span class="md-code-params">()</span> </span>{
      <span class="md-code-keyword">return</span> {
        done: <span class="md-code-keyword">this</span>.items.length === <span class="md-code-number">0</span>,
        value: <span class="md-code-keyword">this</span>.items.shift()
      }
    }
  })
}
<span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">multiplier</span> <span class="md-code-params">(value)</span> </span>{
  <span class="md-code-keyword">yield</span> value * <span class="md-code-number">2</span>
  <span class="md-code-keyword">yield</span> value * <span class="md-code-number">3</span>
  <span class="md-code-keyword">yield</span> value * <span class="md-code-number">4</span>
  <span class="md-code-keyword">yield</span> value * <span class="md-code-number">5</span>
}
<span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">trailmix</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-number">0</span>
  <span class="md-code-keyword">yield</span>* [<span class="md-code-number">1</span>, <span class="md-code-number">2</span>]
  <span class="md-code-keyword">yield</span>* <mark class="md-mark md-code-mark">[...multiplier(<span class="md-code-number">2</span>)]</mark>
  <span class="md-code-keyword">yield</span>* multiplier(<span class="md-code-number">3</span>)
  <span class="md-code-keyword">yield</span>* <mark class="md-mark md-code-mark">foo</mark>
}
<span class="md-code-built_in">console</span>.log([...trailmix()])
<span class="md-code-comment">// &lt;- [0, 1, 2, <mark class="md-mark md-code-mark">4, 6, 8, 10</mark>, 6, 9, 12, 15, <mark class="md-mark md-code-mark">&apos;p&apos;, &apos;o&apos;, &apos;n&apos;, &apos;y&apos;, &apos;f&apos;, &apos;o&apos;, &apos;o&apos;</mark>]</span>
</code></pre> <p>You could also iterate the sequence by hand, calling <code class="md-code md-code-inline">.next()</code>. This approach gives you the most control over the iteration, but it&#x2019;s also the most involved. There&#x2019;s a few features you can leverage here that give you even more control over the iteration.</p> <h2 id="iterating-over-generators-by-hand">Iterating Over Generators by Hand</h2> <p>Besides iterating over <code class="md-code md-code-inline">trailmix</code> as we&#x2019;ve already covered, using <code class="md-code md-code-inline">[...trailmix()]</code>, <code class="md-code md-code-inline">for value of trailmix()</code>, and <code class="md-code md-code-inline">Array.from(trailmix())</code>, we could use the generator returned by <code class="md-code md-code-inline">trailmix()</code> directly, and iterate over that. But <code class="md-code md-code-inline">trailmix</code> was an overcomplicated showcase of <code class="md-code md-code-inline">yield*</code>, let&#x2019;s go back to the <em>side-effects</em> <code class="md-code md-code-inline">generator</code> for this one.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">generator</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;p&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;o&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;n&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;y&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;f&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;o&apos;</span>)
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;o&apos;</span>
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;!&apos;</span>)
}
<span class="md-code-keyword">var</span> g = generator()
<span class="md-code-keyword">while</span> (<span class="md-code-literal">true</span>) {
  <span class="md-code-keyword">let</span> item = <mark class="md-mark md-code-mark">g.next()</mark>
  <span class="md-code-keyword">if</span> (item.done) {
    <span class="md-code-keyword">break</span>
  }
  <span class="md-code-built_in">console</span>.log(item.value)
}
</code></pre> <p>Just like we <a href="http://localhost:3000/articles/es6-iterators-in-depth" aria-label="ES6 Iterators in Depth on Pony Foo">learned yesterday</a>, any items returned by an iterator will have a <code class="md-code md-code-inline">done</code> property that indicates whether the sequence has reached its end, and a <code class="md-code md-code-inline">value</code> indicating the current value in the sequence.</p> <blockquote> <p>If you&#x2019;re confused as to <strong>why the <code class="md-code md-code-inline">&apos;!&apos;</code> is printed</strong> even though there are no more <code class="md-code md-code-inline">yield</code> expressions after it, that&#x2019;s because <code class="md-code md-code-inline">g.next()</code> doesn&#x2019;t know that. The way it works is that each time its called, it executes the method until a <code class="md-code md-code-inline">yield</code> expression is reached, emits its value and <em>suspends execution</em>. The next time <code class="md-code md-code-inline">g.next()</code> is called, _execution is resumed _from where it left off <em>(the last <code class="md-code md-code-inline">yield</code> expression)</em>, until the next <code class="md-code md-code-inline">yield</code> expression is reached. When no <code class="md-code md-code-inline">yield</code> expression is reached, the generator returns <code class="md-code md-code-inline">{ done: true }</code>, signaling that the sequence has ended. At this point, the <code class="md-code md-code-inline">console.log(&apos;!&apos;)</code> statement has been already executed, though.</p> <p>It&#x2019;s also worth noting that <strong>context is preserved</strong> across suspensions and resumptions. That means generators can be stateful. Generators are, in fact, the underlying implementation for <code class="md-code md-code-inline">async</code>/<code class="md-code md-code-inline">await</code> semantics coming in ES7.</p> </blockquote> <p>Whenever <code class="md-code md-code-inline">.next()</code> is called on a generator, there&#x2019;s four &#x201C;events&#x201D; that will suspend execution in the generator, returning an <em><code class="md-code md-code-inline">IteratorResult</code></em> to the caller of <code class="md-code md-code-inline">.next()</code>.</p> <ul> <li>A <code class="md-code md-code-inline">yield</code> expression returning the <em>next</em> value in the sequence</li> <li>A <code class="md-code md-code-inline">return</code> statement returning the <em>last</em> value in the sequence</li> <li>A <code class="md-code md-code-inline">throw</code> statement halts execution in the generator entirely</li> <li>Reaching the end of the generator function signals <code class="md-code md-code-inline">{ done: true }</code></li> </ul> <p>Once the <code class="md-code md-code-inline">g</code> generator ended iterating over a sequence, subsequent calls to <code class="md-code md-code-inline">g.next()</code> will have no effect and just return <code class="md-code md-code-inline">{ done: true }</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">generator</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;only&apos;</span>
}
<span class="md-code-keyword">var</span> g = generator()
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: &apos;only&apos; }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true }</span>
</code></pre> <h2 id="generators-the-del-weird-del-ins-awesome-ins-parts">Generators: The <del>Weird</del> <ins><em>Awesome</em></ins> Parts</h2> <p>Generator objects come with a couple more methods besides <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/next" target="_blank" aria-label="Generator.prototype.next() on MDN"><code class="md-code md-code-inline">.next</code></a>. These are <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/return" target="_blank" aria-label="Generator.prototype.return() on MDN"><code class="md-code md-code-inline">.return</code></a> and <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw" target="_blank" aria-label="Generator.prototype.throw() on MDN"><code class="md-code md-code-inline">.throw</code></a>. We&#x2019;ve already covered <code class="md-code md-code-inline">.next</code> extensively, but not quite. You could also use <code class="md-code md-code-inline">.next(value)</code> to send values <em>into the generator</em>.</p> <p>Let&#x2019;s make <strong>a magic 8-ball generator</strong>. First off, you&#x2019;ll need some answers. Wikipedia obliges, yielding <a href="https://en.wikipedia.org/wiki/Magic_8-Ball#Possible_answers" target="_blank" aria-label="Magic 8 Ball Possible Answers on Wikipedia">20 possible answers</a> for our magic 8-ball.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">var answers = [
  `It is certain`, `It is decidedly so`, `Without a doubt`,
  `Yes definitely`, `You may rely on it`, `As I see it, yes`,
  `Most likely`, `Outlook good`, `Yes`, `Signs point to yes`,
  `Reply hazy try again`, `Ask again later`, `Better not tell you now`,
  `Cannot predict now`, `Concentrate and ask again`,
  `Don&apos;t count on it`, `My reply is no`, `My sources say no`,
  `Outlook not so good`, `Very doubtful`
]
function answer () {
  return answers[Math.floor(Math.random() * answers.length)]
}
</code></pre> <p>The following generator function can act as a <em>&#x201C;genie&#x201D;</em> that answers any questions you might have for them. Note how we discard the first result from <code class="md-code md-code-inline">g.next()</code>. That&#x2019;s because the first call to <code class="md-code md-code-inline">.next</code> enters the generator and there&#x2019;s no <code class="md-code md-code-inline">yield</code> expression waiting to capture the <code class="md-code md-code-inline">value</code> from <code class="md-code md-code-inline">g.next(value)</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">chat</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">while</span> (<span class="md-code-literal">true</span>) {
    <span class="md-code-keyword">let</span> question = <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Genie] &apos;</span> + answer()
    <span class="md-code-built_in">console</span>.log(question)
  }
}
<span class="md-code-keyword">var</span> g = chat()
<mark class="md-mark md-code-mark">g.next()</mark>
<span class="md-code-built_in">console</span>.log(g.next(<span class="md-code-string">&apos;[Me] Will ES6 die a painful death?&apos;</span>).value)
<span class="md-code-comment">// &lt;- &apos;[Me] Will ES6 die a painful death?&apos;</span>
<span class="md-code-comment">// &lt;- &apos;[Genie] My sources say no&apos;</span>
<span class="md-code-built_in">console</span>.log(g.next(<span class="md-code-string">&apos;[Me] How youuu doing?&apos;</span>).value)
<span class="md-code-comment">// &lt;- &apos;[Me] How youuu doing?&apos;</span>
<span class="md-code-comment">// &lt;- &apos;[Genie] Concentrate and ask again&apos;</span>
</code></pre> <p>Randomly dropping <code class="md-code md-code-inline">g.next()</code> feels like a very dirty coding practice, though. What else could we do? We could flip responsibilities around.</p> <h3 id="inversion-of-control">Inversion of Control</h3> <p>We could have the Genie be in control, and have the generator ask the questions. How would that look like? At first, you might think that the code below is unconventional, but in fact, most libraries built around generators work by inverting responsibility.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">chat</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] Will ES6 die a painful death?&apos;</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] How youuu doing?&apos;</span>
}
<span class="md-code-keyword">var</span> g = chat()
<span class="md-code-keyword">while</span> (<span class="md-code-literal">true</span>) {
  <span class="md-code-keyword">let</span> question = g.next()
  <span class="md-code-keyword">if</span> (question.done) {
    <span class="md-code-keyword">break</span>
  }
  <span class="md-code-built_in">console</span>.log(question.value)
  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;[Genie] &apos;</span> + answer())
  <span class="md-code-comment">// &lt;- &apos;[Me] Will ES6 die a painful death?&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;[Genie] Very doubtful&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;[Me] How youuu doing?&apos;</span>
  <span class="md-code-comment">// &lt;- &apos;[Genie] My reply is no&apos;</span>
}
</code></pre> <p>You would expect the <strong>generator to do the heavy lifting</strong> of an iteration, but in fact generators make it easy to iterate over things by suspending execution of themselves &#x2013; and deferring the heavy lifting. That&#x2019;s one of the most powerful aspects of generators. Suppose now that the iterator is a <code class="md-code md-code-inline">genie</code> method in a library, like so:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">genie</span> <span class="md-code-params">(questions)</span> </span>{
  <span class="md-code-keyword">var</span> g = questions()
  <span class="md-code-keyword">while</span> (<span class="md-code-literal">true</span>) {
    <span class="md-code-keyword">let</span> question = g.next()
    <span class="md-code-keyword">if</span> (question.done) {
      <span class="md-code-keyword">break</span>
    }
    <span class="md-code-built_in">console</span>.log(question.value)
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;[Genie] &apos;</span> + answer())
  }
}
</code></pre> <p>To use it, all you&#x2019;d have to do is pass in a simple generator like the one we just made.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">genie(<span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">questions</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] Will ES6 die a painful death?&apos;</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] How youuu doing?&apos;</span>
})
</code></pre> <p>Compare that to the generator we had before, where questions were sent to the generator instead of the other way around. See how much more complicated the logic would have to be to achieve the same goal? Letting the library deal with the flow control means you can <strong>just worry about the <em>thing</em> you want to iterate</strong> over, and you can <strong>delegate <em>how</em> to iterate over it</strong>. But yes, it does mean your code now has an asterisk in it. <em>Weird.</em></p> <h3 id="dealing-with-asynchronous-flows">Dealing with asynchronous flows</h3> <p>Imagine now that the <code class="md-code md-code-inline">genie</code> library gets its magic 8-ball answers from an API. How does that look then? Probably something like the snippet below. Assume the <a href="https://github.com/Raynos/xhr" target="_blank" aria-label="Raynos/xhr on GitHub"><code class="md-code md-code-inline">xhr</code></a> pseudocode call always yields JSON responses like <code class="md-code md-code-inline">{ answer: &apos;No&apos; }</code>. Keep in mind this is a simple example that just processes each question in series. You could put together different and more complex flow control algorithms depending on what you&#x2019;re looking for.</p> <p>This is just a demonstration of the sheer power of generators.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">genie</span> <span class="md-code-params">(questions)</span> </span>{
  <span class="md-code-keyword">var</span> g = questions()
  pull()
  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">pull</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">let</span> question = <mark class="md-mark md-code-mark">g.next()</mark>
    <span class="md-code-keyword">if</span> (question.done) {
      <span class="md-code-keyword">return</span>
    }
    ask(question.value, pull)
  }
  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">ask</span> <span class="md-code-params">(q, next)</span> </span>{
    <mark class="md-mark md-code-mark">xhr(<span class="md-code-string">&apos;https://computer.genie/?q=&apos;</span> + <span class="md-code-built_in">encodeURIComponent</span>(q), got)</mark>
    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">got</span> <span class="md-code-params">(err, res, body)</span> </span>{
      <span class="md-code-keyword">if</span> (err) {
        <span class="md-code-comment">// todo</span>
      }
      <span class="md-code-built_in">console</span>.log(q)
      <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;[Genie] &apos;</span> + body.answer)
      next()
    }
  }
}
</code></pre> <p><sub>See <a href="http://buff.ly/1UimWsZ" target="_blank" aria-label="Babel REPL of async generator for genie responses">this link for a live demo</a> on the Babel REPL</sub></p> <p>Even though we&#x2019;ve just made our <code class="md-code md-code-inline">genie</code> method asynchronous and are now using an API to fetch responses to the user&#x2019;s questions, the way the consumer uses the <code class="md-code md-code-inline">genie</code> library by passing a <code class="md-code md-code-inline">questions</code> generator function <em>remains unchanged!</em> That&#x2019;s awesome.</p> <p>We haven&#x2019;t handled the case for an <code class="md-code md-code-inline">err</code> coming out of the API. That&#x2019;s inconvenient. What can we do about that one?</p> <h3 id="throwing-at-a-generator">Throwing <em>at</em> a Generator</h3> <p>Now that we&#x2019;ve figured out that the most important aspect of generators is <em>actually the control flow code</em> that decides when to call <code class="md-code md-code-inline">g.next()</code>, we can look at the other two methods and actually understand their purpose. Before shifting our thinking into <em>&#x201C;the generator defines <strong>what</strong> to iterate over, not the <strong>how</strong>&#x201D;</em>, we would&#x2019;ve been hard pressed to find a user case for <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw" target="_blank" aria-label="Generator.prototype.throw() on MDN"><code class="md-code md-code-inline">g.throw</code></a>. Now however it seems immediately obvious. The flow control that leverages a generator needs to be able to tell the generator that&#x2019;s yielding the sequence to be iterated when something goes wrong processing an item in the sequence.</p> <p>In the case of our <code class="md-code md-code-inline">genie</code> flow, that is now using <a href="https://github.com/Raynos/xhr" target="_blank" aria-label="Raynos/xhr on GitHub"><code class="md-code md-code-inline">xhr</code></a>, we may experience network issues and be unable to continue processing items, or we may want to warn the user about unexpected errors. Here&#x2019;s how, we simply add <code class="md-code md-code-inline">g.throw(error)</code> in our control flow code.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">genie</span> <span class="md-code-params">(questions)</span> </span>{
  <span class="md-code-keyword">var</span> g = questions()
  pull()
  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">pull</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">let</span> question = g.next()
    <span class="md-code-keyword">if</span> (question.done) {
      <span class="md-code-keyword">return</span>
    }
    ask(question.value, pull)
  }
  <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">ask</span> <span class="md-code-params">(q, next)</span> </span>{
    xhr(<span class="md-code-string">&apos;https://computer.genie/?q=&apos;</span> + <span class="md-code-built_in">encodeURIComponent</span>(q), got)
    <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">got</span> <span class="md-code-params">(err, res, body)</span> </span>{
      <span class="md-code-keyword">if</span> (err) {
        <mark class="md-mark md-code-mark">g.throw(err)</mark>
      }
      <span class="md-code-built_in">console</span>.log(q)
      <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;[Genie] &apos;</span> + body.answer)
      next()
    }
  }
}
</code></pre> <p>The <em>user code</em> is still unchanged, though. In between <code class="md-code md-code-inline">yield</code> statements it may throw errors now. You could use <code class="md-code md-code-inline">try</code>/<code class="md-code md-code-inline">catch</code> blocks to address those issues. If you do this, execution will be able to resume. The good thing is that this is up to the user, it&#x2019;s still perfectly sequential on their end, and they can leverage <code class="md-code md-code-inline">try</code>/<code class="md-code md-code-inline">catch</code> semantics just like in high-school.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">genie(<span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">questions</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">try</span> {
    <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] Will ES6 die a painful death?&apos;</span>
  } <span class="md-code-keyword">catch</span> (e) {
    <span class="md-code-built_in">console</span>.error(<span class="md-code-string">&apos;Error&apos;</span>, e.message)
  }
  <span class="md-code-keyword">try</span> {
    <span class="md-code-keyword">yield</span> <span class="md-code-string">&apos;[Me] How youuu doing?&apos;</span>
  } <span class="md-code-keyword">catch</span> (e) {
    <span class="md-code-built_in">console</span>.error(<span class="md-code-string">&apos;Error&apos;</span>, e.message)
  }
})
</code></pre> <h3 id="returning-on-behalf-of-a-generator">Returning on Behalf of a Generator</h3> <p>Usually not as interesting in asynchronous control flow mechanisms in general, the <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/return" target="_blank" aria-label="Generator.prototype.return() on MDN"><code class="md-code md-code-inline">g.return()</code></a> method allows you to resume execution inside a generator function, much like <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator/throw" target="_blank" aria-label="Generator.prototype.throw() on MDN"><code class="md-code md-code-inline">g.throw()</code></a> did moments earlier. The key difference is that <code class="md-code md-code-inline">g.return()</code> won&#x2019;t result in an exception at the generator level, although <strong>it will end the sequence.</strong></p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">numbers</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-number">1</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">2</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">3</span>
}
<span class="md-code-keyword">var</span> g = numbers()
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 1 }</span>
<span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">g.return()</mark>)
<span class="md-code-comment">// &lt;- { done: true }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true }, <mark class="md-mark md-code-mark">as we know</mark></span>
</code></pre> <p>You could also return a <code class="md-code md-code-inline">value</code> using <code class="md-code md-code-inline">g.return(value)</code>, and the resulting <code class="md-code md-code-inline">IteratorResult</code> will contain said <code class="md-code md-code-inline">value</code>. This is equivalent to having <code class="md-code md-code-inline">return value</code> somewhere in the generator function. You should be careful there though &#x2013; as neither <code class="md-code md-code-inline">for..of</code>, <code class="md-code md-code-inline">[...generator()]</code>, nor <code class="md-code md-code-inline">Array.from(generator())</code> include the <code class="md-code md-code-inline">value</code> in the <code class="md-code md-code-inline">IteratorResult</code> that signals <code class="md-code md-code-inline">{ done: true }</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">numbers</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-number">1</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">2</span>
  <mark class="md-mark md-code-mark">return <span class="md-code-number">3</span></mark>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">4</span>
}
<span class="md-code-built_in">console</span>.log([...numbers()])
<span class="md-code-comment">// &lt;- <mark class="md-mark md-code-mark">[1, 2]</mark></span>
<span class="md-code-built_in">console</span>.log(<span class="md-code-built_in">Array</span>.from(numbers()))
<span class="md-code-comment">// &lt;- [1, 2]</span>
<span class="md-code-keyword">for</span> (<span class="md-code-keyword">let</span> n of numbers()) {
  <span class="md-code-built_in">console</span>.log(n)
  <span class="md-code-comment">// &lt;- 1</span>
  <span class="md-code-comment">// &lt;- 2</span>
}
<span class="md-code-keyword">var</span> g = numbers()
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 1 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 2 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true, <mark class="md-mark md-code-mark">value: 3</mark> }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true }</span>
</code></pre> <p>Using <code class="md-code md-code-inline">g.return</code> is no different in this regard, think of it as the programmatic equivalent of what we just did.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">numbers</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-number">1</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">2</span>
  <span class="md-code-keyword">return</span> <span class="md-code-number">3</span>
  <span class="md-code-keyword">yield</span> <span class="md-code-number">4</span>
}
<span class="md-code-keyword">var</span> g = numbers()
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 1 }</span>
<span class="md-code-built_in">console</span>.log(g.return(<span class="md-code-number">5</span>))
<span class="md-code-comment">// &lt;- { done: true, value: 5 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true }</span>
</code></pre> <p>You can avoid the impending sequence termination, <a href="http://www.2ality.com/2015/03/es6-generators.html" target="_blank" aria-label="ES6 generators in depth by Dr. Axel Rauschmayer">as Axel points out</a>, if the code in the generator function when <code class="md-code md-code-inline">g.return()</code> got called is wrapped in <code class="md-code md-code-inline">try</code>/<code class="md-code md-code-inline">finally</code>. Once the <code class="md-code md-code-inline">yield</code> expressions in the <code class="md-code md-code-inline">finally</code> block are over, the sequence <em>will</em> end with the <code class="md-code md-code-inline">value</code> passed to <code class="md-code md-code-inline">g.return(value)</code></p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span>* <span class="md-code-title">numbers</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">yield</span> <span class="md-code-number">1</span>
  <span class="md-code-keyword">try</span> {
    <span class="md-code-keyword">yield</span> <span class="md-code-number">2</span>
  } <span class="md-code-keyword">finally</span> {
    <span class="md-code-keyword">yield</span> <span class="md-code-number">3</span>
    <span class="md-code-keyword">yield</span> <span class="md-code-number">4</span>
  }
  <span class="md-code-keyword">yield</span> <span class="md-code-number">5</span>
}
<span class="md-code-keyword">var</span> g = numbers()
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 1 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 2 }</span>
<span class="md-code-built_in">console</span>.log(<mark class="md-mark md-code-mark">g.return(<span class="md-code-number">6</span>)</mark>)
<span class="md-code-comment">// &lt;- { done: false, value: 3 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: false, value: 4 }</span>
<span class="md-code-built_in">console</span>.log(g.next())
<span class="md-code-comment">// &lt;- { done: true, <mark class="md-mark md-code-mark">value: 6</mark> }</span>
</code></pre> <p>That&#x2019;s all there is to know when it comes to generators <em>in terms of functionality.</em></p> <h2 id="use-cases-for-es6-generators">Use Cases for ES6 Generators</h2> <p>At this point in the article you should feel comfortable with the concepts of iterators, iterables, and generators in ES6. If you feel like reading more on the subject, I highly recommend you go over <a href="http://www.2ality.com/2015/03/es6-generators.html" target="_blank" aria-label="ES6 generators in depth by Dr. Axel Rauschmayer">Axel&#x2019;s article on generators</a>, as he put together an amazing write-up on use cases for generators just <em>a few months ago</em>.</p></div>
