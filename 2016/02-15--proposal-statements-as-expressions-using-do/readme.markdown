<div></div>

<h1>Proposal: &#x201C;Statements as Expressions&#x201D; using <code class="md-code md-code-inline">do</code></h1>

<p><kbd>ecmascript</kbd> <kbd>do-expressions</kbd> <kbd>proposal-draft</kbd></p>

<blockquote><p>A proposal for <code>do</code> statements has been classified as <strong>Stage 0</strong> for a while, and it might be an interesting solution for some problems we can find in JavaScript.</p>
</blockquote>

<div><p>A proposal for <code class="md-code md-code-inline">do</code> statements has been classified as <strong>Stage 0</strong> for a while, and it might be an interesting solution for some problems we can find in JavaScript.</p></div>

<div></div>

<div><p>When JavaScript expressions are evaluated, they produce a single value.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-number">3</span> * <span class="md-code-number">10</span>
<span class="md-code-comment">// &lt;- 30</span>
</code></pre> <p>If we want to add a condition in an expression, we need to use ternary expressions or logical operators. The following example displays both alternatives, although the former is usually preferred.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-number">1</span>, -<span class="md-code-number">1</span>, -<span class="md-code-number">0.5</span>].map(x =&gt; <mark class="md-mark md-code-mark">x &gt; <span class="md-code-number">0</span> ? x * <span class="md-code-number">10</span> : -x * <span class="md-code-number">10</span></mark>)
<span class="md-code-comment">// &lt;- [10, 10, 5]</span>
[<span class="md-code-number">1</span>, -<span class="md-code-number">1</span>, -<span class="md-code-number">0.5</span>].map(x =&gt; <mark class="md-mark md-code-mark">x &gt; <span class="md-code-number">0</span> &amp;&amp; x * <span class="md-code-number">10</span> || -x * <span class="md-code-number">10</span></mark>)
<span class="md-code-comment">// &lt;- [10, 10, 5]</span>
</code></pre> <p>It can be confusing to create side effects, but you can achieve that through use of commas.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">sideEffect(), <span class="md-code-number">3</span> * <span class="md-code-number">10</span>
<span class="md-code-comment">// &lt;- 30</span>
</code></pre> <p>It&#x2019;s not possible to declare variables you might need for temporary storage <em>within an expression</em>. As such, you typically extract these variables into the enclosing scope. More often than not, that&#x2019;s better for readability anyways, so it doesn&#x2019;t have a hugely negative impact.</p></div>

<div><h1 id="using-do">Using <code class="md-code md-code-inline">do</code></h1> <p>The <code class="md-code md-code-inline">do</code> expression proposal lets you write a block of statements to be evaluated. The &#x201C;completion value&#x201D; is returned as the result of a <code class="md-code md-code-inline">do</code> expression. In the following example, there&#x2019;s just a <code class="md-code md-code-inline">3 * 10</code> expression in our block, so that&#x2019;s our completion value, and <code class="md-code md-code-inline">30</code> is returned.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><mark class="md-mark md-code-mark">do {</mark> <span class="md-code-number">3</span> * <span class="md-code-number">10</span> <mark class="md-mark md-code-mark">}</mark> <span class="md-code-comment">// just an expression</span>
<span class="md-code-comment">// &lt;- 30</span>
</code></pre> <p>The following bit of code is equivalent to the two <code class="md-code md-code-inline">.map</code> examples we saw earlier. In this case, we use a <code class="md-code md-code-inline">do</code> expression, allowing us to use <code class="md-code md-code-inline">if</code> and <code class="md-code md-code-inline">else</code> instead of ternary or logical operators.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">[<span class="md-code-number">1</span>, -<span class="md-code-number">1</span>, -<span class="md-code-number">0.5</span>].map(x =&gt; <mark class="md-mark md-code-mark">do { <span class="md-code-keyword">if</span> (x &gt; <span class="md-code-number">0</span>) {</mark> x * <span class="md-code-number">10</span> <mark class="md-mark md-code-mark">} <span class="md-code-keyword">else</span> {</mark> -x * <span class="md-code-number">10</span> <mark class="md-mark md-code-mark">} }</mark>)
<span class="md-code-comment">// &lt;- [10, 10, 5]</span>
</code></pre> <p>Side effects in <code class="md-code md-code-inline">do</code> expressions become easier to read, and we are able to declare variables. In the following example we&#x2019;re purposely missing a <code class="md-code md-code-inline">return</code> statement, as <code class="md-code md-code-inline">do</code> expressions already implicitly return, as we saw in the case of the <code class="md-code md-code-inline">if</code> / <code class="md-code md-code-inline">else</code> example. Naturally, <code class="md-code md-code-inline">const</code> and <code class="md-code md-code-inline">let</code> variables declared inside a <code class="md-code md-code-inline">do</code> block are scoped to that block, while <code class="md-code md-code-inline">var</code> variables are scoped to the containing function.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> data = <span class="md-code-keyword">do</span> {
  <span class="md-code-keyword">const</span> data = pullSomeData()
  doSomethingElse() <span class="md-code-comment">// sideEffect</span>
  data
}
</code></pre> <h1 id="using-do-today">Using <code class="md-code md-code-inline">do</code> Today</h1> <p>It&#x2019;s easy, there&#x2019;s <a href="https://github.com/babel/babel/tree/master/packages/babel-plugin-syntax-do-expressions" target="_blank" aria-label="babel/packages/babel-plugin-syntax-do-expressions on GitHub">a Babel plugin</a> we can use.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm install --save-dev babel-plugin-syntax-do-expressions
</code></pre> <p>Then add the following to your <code class="md-code md-code-inline">.babelrc</code> file or the <code class="md-code md-code-inline">babel</code> property in <code class="md-code md-code-inline">package.json</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-json">{
  &quot;<span class="md-code-attribute">plugins</span>&quot;: <span class="md-code-value">[<span class="md-code-string">&quot;syntax-do-expressions&quot;</span>]
</span>}
</code></pre> <p><em>That&#x2019;s it.</em></p> <p>Whenever you run the Babel CLI, it&#x2019;ll understand <code class="md-code md-code-inline">do</code> expressions.</p> <h1 id="conditionals-in-jsx">Conditionals in JSX</h1> <p>A while back I wrote about <a href="https://ponyfoo.com/articles/react-jsx-and-es6-the-weird-parts#using-conditionals-in-your-view-components" aria-label="Using conditionals in your JSX view components"><em>&#x201C;the weird parts&#x201D;</em> of using JSX</a> &#x2013; the JavaScript syntax extension Facebook built to help you write templates for React apps. Back then, I mentioned how sometimes you have to write code like the following when you want to conditionally render a piece of markup.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">return</span> (
  <span><span class="md-code-tag">&lt;<span class="md-code-title">nav</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">Home</span> /&gt;</span>
    { loggedIn &amp;&amp; <span class="md-code-tag">&lt;<span class="md-code-title">LogoutButton</span> /&gt;</span> || <span class="md-code-tag">&lt;<span class="md-code-title">LoginButton</span> /&gt;</span> }
  <span class="md-code-tag">&lt;/<span class="md-code-title">nav</span>&gt;</span>
);
</span></code></pre> <p>With <code class="md-code md-code-inline">do</code> expressions, you could get rid of the weird-looking and <em>oft-confusing</em> logical operators. This makes the code easier to read and saves you from having to deal with falsy expressions like <code class="md-code md-code-inline">0</code> or <code class="md-code md-code-inline">&apos;&apos;</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">return</span> (
  <span><span class="md-code-tag">&lt;<span class="md-code-title">nav</span>&gt;</span>
    <span class="md-code-tag">&lt;<span class="md-code-title">Home</span> /&gt;</span>
    {
      <mark class="md-mark md-code-mark">do {</mark>
        <mark class="md-mark md-code-mark">if (loggedIn) {</mark>
          <span class="md-code-tag">&lt;<span class="md-code-title">LogoutButton</span> /&gt;</span>
        <mark class="md-mark md-code-mark">} else {</mark>
          <span class="md-code-tag">&lt;<span class="md-code-title">LoginButton</span> /&gt;</span>
        <mark class="md-mark md-code-mark">}</mark>
      <mark class="md-mark md-code-mark">}</mark>
    }
  <span class="md-code-tag">&lt;/<span class="md-code-title">nav</span>&gt;</span>
)
</span></code></pre> <p>When there&#x2019;s larger pieces of markup it becomes more elegant to be able to use statements instead of expressions &#x2013; and <code class="md-code md-code-inline">do</code> let&#x2019;s you <code class="md-code md-code-inline">do</code> that. The <code class="md-code md-code-inline">do</code> syntax makes it easier to read conditionals, allows you to use variables, and makes it clear when part of an expression is a side effect.</p></div>
