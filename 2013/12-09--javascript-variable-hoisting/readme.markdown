<h1>JavaScript Variable Hoisting</h1>

<p><kbd>js</kbd> <kbd>hoisting</kbd></p>

<blockquote><p>A large number of JavaScript interview questions, if not most of them, can be answered with an understanding of scoping, <a href="https://ponyfoo.com/2013/12/04/where-does-this-keyword-come-from">how <code>this</code> works</a>, and hoisting.</p><p>You might be &#x2026;</p></blockquote>

<div><p>A large number of JavaScript interview questions, if not most of them, can be answered with an understanding of scoping, <a href="https://ponyfoo.com/2013/12/04/where-does-this-keyword-come-from">how <code class="md-code md-code-inline">this</code> works</a>, and hoisting.</p></div>

<div></div>

<div><p>You might be expecting the method to print <code class="md-code md-code-inline">&apos;number&apos;</code> first, and <code class="md-code md-code-inline">2</code> afterwards, or maybe <code class="md-code md-code-inline">3</code>? Try running it! Why does it print <code class="md-code md-code-inline">&apos;undefined&apos;</code> and then <code class="md-code md-code-inline">undefined</code>? Well, hello hoisting! It&#x2019;ll be easier for you to picture it if I re-arrange the code to how it ends up after hoisting takes place. Let&#x2019;s have a look.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> value = <span class="md-code-number">2</span>;

test();

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">test</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">typeof</span> value);
    <span class="md-code-built_in">console</span>.log(value);
    <span class="md-code-keyword">var</span> value = <span class="md-code-number">3</span>;
}
</code></pre></div>

<div><p>Enter <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Scope_Cheatsheet#Hoisting" target="_blank" aria-label="Variable Hosting on MDN">JavaScript variable hoisting</a>, and your code will actually end up looking like below. Hoisting <em>basically moves variable declarations</em> to the top of the scope those variables belong to. However, assignments stay where they are! Function declarations are hoisted only if they&#x2019;re not part of an assignment statement.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> value;

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">test</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">var</span> value;
    <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">typeof</span> value);
    <span class="md-code-built_in">console</span>.log(value);
    value = <span class="md-code-number">3</span>;
}

value = <span class="md-code-number">2</span>;

test();
</code></pre> <p>So, that&#x2019;s why: the <code class="md-code md-code-inline">value</code> declaration at the end of the <code class="md-code md-code-inline">test</code> function actually got hoisted to the top of the scope, and also the reason why <code class="md-code md-code-inline">test</code> didn&#x2019;t meet us with a <code class="md-code md-code-inline">TypeError</code> exception warning us about <code class="md-code md-code-inline">undefined</code> not being a function. Keep in mind that if we used the variable form of declaring the <code class="md-code md-code-inline">test</code> function, we would in fact have gotten that error, because although <code class="md-code md-code-inline">var test</code> would&#x2019;ve been hoisted, the assignment wouldn&#x2019;t have been, effectively becoming the following:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> value;
<span class="md-code-keyword">var</span> test;

value = <span class="md-code-number">2</span>;

test();

test = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
    <span class="md-code-keyword">var</span> value;
    <span class="md-code-built_in">console</span>.log(<span class="md-code-keyword">typeof</span> value);
    <span class="md-code-built_in">console</span>.log(value);
    value = <span class="md-code-number">3</span>;
};
</code></pre> <p>The above wouldn&#x2019;t work as expected, because <code class="md-code md-code-inline">test</code> isn&#x2019;t defined by the time we want to invoke it.</p> <p><img alt="hoisting.png" title="Variable hoisting in action" class="" src="https://i.imgur.com/eGT7oTe.png"></p> <h3 id="in-the-real-world">In the real world</h3> <p>Below is a real bug I had to track down once upon a time. Here, the issue was that we were declaring a <code class="md-code md-code-inline">path</code> variable in the inner scope, shadowing the <code class="md-code md-code-inline">path</code> in the outer scope. Due to hoisting, the inner <code class="md-code md-code-inline">path</code> would be <code class="md-code md-code-inline">undefined</code> if <code class="md-code md-code-inline">!something</code>, resulting in unexpected behavior. This goes to show how much better it would&#x2019;ve been to stick to a pattern where we &#x201C;hoist&#x201D; variables ourselves, rather than letting the language to do it for us. Hoisting code would improve the visibility of potential issues such as the one depicted below.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> path = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;path&apos;</span>);

<span class="md-code-comment">// ...</span>

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(something)</span> </span>{
  <span class="md-code-comment">// ...</span>

  <span class="md-code-keyword">if</span> (something) {
    <span class="md-code-keyword">var</span> path = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;path&apos;</span>);
    <span class="md-code-comment">// ...</span>
  }

  <span class="md-code-comment">// ...</span>
};
</code></pre> <p>Granted, it&#x2019;s not a very common issue, but it happens!</p></div>
