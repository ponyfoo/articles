<h1>Publishing Node.JS Packages with npm</h1>

<div><kbd>nodejs</kbd> <kbd>npm</kbd></div>

<blockquote><p>Back when I <a href="https://ponyfoo.com/2013/01/18/asset-management-in-node">introduced assetify</a>, I mentioned publishing packages on <a href="https://npmjs.org/" target="_blank">npm</a> is <em>very</em> easy.</p></blockquote>

<div><p>Back when I <a href="https://ponyfoo.com/2013/01/18/asset-management-in-node">introduced assetify</a>, I mentioned publishing packages on <a href="https://npmjs.org/" target="_blank">npm</a> is <em>very</em> easy.</p></div>

<div></div>

<div></div>

<div><p>The first thing you have to do is <strong>identify yourself</strong>, create an account on <strong>npm</strong>:</p> <pre class="md-code-block"><code class="md-code md-lang-bash">$ npm adduser
</code></pre> <p>After that all you have to do is code up a <a href="https://npmjs.org/doc/developers.html" target="_blank" aria-label="package.json specs">package.json</a>, which essentially requires a package <em>name</em> and a <em>version</em> string.</p> <p>Here&#x2019;s an example <strong>package.json</strong>, extracted from <a href="https://github.com/bevacqua/jsn" target="_blank" aria-label="JSN on GitHub">jsn</a>:</p> <pre class="md-code-block"><code class="md-code md-lang-json">{
  &quot;<span class="md-code-attribute">name</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;jsn&quot;</span></span>,
  &quot;<span class="md-code-attribute">description</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;JavaScript Node server-side variable parser&quot;</span></span>,
  &quot;<span class="md-code-attribute">author</span>&quot;: <span class="md-code-value">{
    &quot;<span class="md-code-attribute">name</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;Nicolas Bevacqua&quot;</span></span>,
    &quot;<span class="md-code-attribute">email</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;nicolasbevacqua@gmail.com&quot;</span></span>,
    &quot;<span class="md-code-attribute">url</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;http://www.ponyfoo.com&quot;</span>
  </span>}</span>,
  &quot;<span class="md-code-attribute">version</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;0.0.3&quot;</span></span>,
  &quot;<span class="md-code-attribute">dependencies</span>&quot;: <span class="md-code-value">{
  }</span>,
  &quot;<span class="md-code-attribute">devDependencies</span>&quot;: <span class="md-code-value">{
    &quot;<span class="md-code-attribute">vows</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;0.5.x&quot;</span></span>,
    &quot;<span class="md-code-attribute">mocha</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;*&quot;</span></span>,
    &quot;<span class="md-code-attribute">should</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;*&quot;</span>
  </span>}</span>,
  &quot;<span class="md-code-attribute">repository</span>&quot;: <span class="md-code-value">{
    &quot;<span class="md-code-attribute">type</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;git&quot;</span></span>,
    &quot;<span class="md-code-attribute">url</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;git://github.com/bevacqua/jsn.git&quot;</span>
  </span>}</span>,
  &quot;<span class="md-code-attribute">bugs</span>&quot;: <span class="md-code-value">{
    &quot;<span class="md-code-attribute">url</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;https://github.com/bevacqua/jsn/issues&quot;</span>
  </span>}</span>,
  &quot;<span class="md-code-attribute">licenses</span>&quot;: <span class="md-code-value">[{
    &quot;<span class="md-code-attribute">type</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;MIT&quot;</span>
  </span>}]</span>,
  &quot;<span class="md-code-attribute">main</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;./src/compiler.js&quot;</span></span>,
  &quot;<span class="md-code-attribute">engines</span>&quot;: <span class="md-code-value">{
    &quot;<span class="md-code-attribute">node</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;&gt;= 0.8.x&quot;</span></span>,
    &quot;<span class="md-code-attribute">npm</span>&quot;: <span class="md-code-value"><span class="md-code-string">&quot;&gt;= 1.1.x&quot;</span>
  </span>}
</span>}
</code></pre> <p>The only thing that might be worth mentioning is that you should make sure to specify a <code class="md-code md-code-inline">&quot;main&quot;</code> property, so that the module gets exported appropriately through <em>the entry point you intended</em>.</p> <p>Always make sure to thoroughly test the logic in your packages before unleashing them into the wild. If you happen to publish a <em>faulty image</em> of your package, you <strong>can overwrite</strong> it using the <code class="md-code md-code-inline">--force</code> flag:</p> <pre class="md-code-block"><code class="md-code md-lang-bash">$ npm publish --force
</code></pre> <p>In addition to <strong>.gitignore</strong>, <strong>.npmignore</strong> files can come in handy, here is an example, again from the <strong>jsn</strong> repository on <em>GitHub</em>.</p> <pre class="md-code-block"><code class="md-code">*.md
.DS_Store
.git*
.idea
Makefile
docs/
examples/
support/
test/
</code></pre> <p>Spread the word!</p></div>
