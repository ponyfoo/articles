<div></div>

<h1>Spritesheets, Grunt, and You</h1>

<p><kbd>grunt</kbd> <kbd>build</kbd> <kbd>sprites</kbd></p>

<blockquote><p>If you are using Grunt, you really have <strong>no excuse</strong> not to be using CSS spritesheets. If you aren&#x2019;t using Grunt <em>yet</em>, then you should know that a well thought-out &#x2026;</p></blockquote>

<div><p>If you are using Grunt, you really have <strong>no excuse</strong> not to be using CSS spritesheets. If you aren&#x2019;t using Grunt <em>yet</em>, then you should know that a well thought-out workflow with Grunt will allow you to <em>seamlessly integrate icons together</em> into a spritesheet during your builds.</p></div>

<div></div>

<div><p>In case you&#x2019;ve been living undersea for the last decade, I wonder how you&#x2019;re still breathing. Sprites are <a href="http://en.wikipedia.org/wiki/Sprite_(computer_graphics)" target="_blank">a very old concept</a>, originating in game engines, which were more performant when they just loaded a <em>single graphic</em> containing all the frames needed to render an animation. Today, spritesheets are a well-known improvement over using a single image for each icon in modern websites, but they are still slower to gain ground than they could be.</p> <p>Grunt makes this super-easy for us, and in this article I want to demonstrate exactly how I use it to generate spritesheets.</p></div>

<div><p><img alt="spritesheet.gif" title="An spritesheet, used in Megaman" class="" src="https://i.imgur.com/1ud2mRR.gif"></p> <p>The first thing we&#x2019;ll want to install is <code class="md-code md-code-inline">grunt-spritesmith</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm install --save-dev grunt-spritesmith
</code></pre> <p>Configuring it is kind of tricky to get right, but I&#x2019;ve put together a function to make it easier for us to set it up.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">sprite</span> <span class="md-code-params">(type, short)</span> </span>{
  <span class="md-code-keyword">return</span> {
    src: <span class="md-code-string">&apos;src/img/sprite/&apos;</span> + type + <span class="md-code-string">&apos;/**/*.{png,jpg,gif}&apos;</span>,
    destImg: <span class="md-code-string">&apos;bin/public/img/sprite/&apos;</span> + type + <span class="md-code-string">&apos;.png&apos;</span>,
    destCSS: <span class="md-code-string">&apos;bin/public/css/sprite/&apos;</span> + type + <span class="md-code-string">&apos;.css&apos;</span>,
    imgPath: <span class="md-code-string">&apos;/img/sprite/&apos;</span> + type + <span class="md-code-string">&apos;.png&apos;</span>,
    cssOpts: {
      cssClass: <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(item)</span> </span>{
        <span class="md-code-keyword">var</span> prefix = short ? short + <span class="md-code-string">&apos;-&apos;</span> : <span class="md-code-string">&apos;&apos;</span>;
        <span class="md-code-keyword">return</span> <span class="md-code-string">&apos;.&apos;</span> + prefix + item.name + <span class="md-code-string">&apos;:before&apos;</span>;
      }
    },
    engine: <span class="md-code-string">&apos;gm&apos;</span>,
      imgOpts: {
        format: <span class="md-code-string">&apos;png&apos;</span>
      }
  };
}
</code></pre> <p>Say we want to set up a <strong>&#x201C;tools&#x201D;</strong> spritesheet, with a bunch of tool icons we have. We can simply configure Grunt like below.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">grunt.initConfig({ 
  sprite: {
    tools: sprite(<span class="md-code-string">&apos;tools&apos;</span>, <span class="md-code-string">&apos;tl&apos;</span>)
  }
});
</code></pre> <p>Lastly, we&#x2019;ll need a few extra tweaks to our CSS so that it works in a <code class="md-code md-code-inline">:before</code> pseudo-element.</p> <pre class="md-code-block"><code class="md-code md-lang-css"><span class="md-code-class">.icon</span><span class="md-code-pseudo">:before</span> <span class="md-code-rules">{
  <span><span class="md-code-attribute">vertical-align</span>:<span class="md-code-value"> bottom</span></span>;
  <span><span class="md-code-attribute">display</span>:<span class="md-code-value"> inline-block</span></span>;
  <span><span class="md-code-attribute">content</span>:<span class="md-code-value"> <span class="md-code-string">&apos;&apos;</span></span></span>;
<span>}</span></span>
</code></pre> <p>That&#x2019;s it, we&#x2019;re now able to render sprited icons in our HTML, effortlessly.</p> <pre class="md-code-block"><code class="md-code md-lang-xml"><span class="md-code-tag">&lt;<span class="md-code-title">span</span> <span class="md-code-attribute">class</span>=<span class="md-code-value">&quot;icon tl-screw&quot;</span>&gt;</span><span class="md-code-tag">&lt;/<span class="md-code-title">span</span>&gt;</span>
</code></pre> <p>The tremendous upside is that we can drag and drop new icons into our source folder, and immediately build a spritesheet again. Something that used to take <em>pestering the designer</em>, or considerable effort on <strong>Paint</strong> on our part, is now just a matter of typing <code class="md-code md-code-inline">grunt sprite</code> into the terminal.</p> <p><img alt="icons-directory.png" title="Just drag and drop!" class="" src="https://i.imgur.com/5pRLV9c.png"></p> <p>Fully working <a href="https://github.com/bevacqua/grunt-spriting-example" target="_blank" aria-label="grunt-spriting-example on GitHub">source code here</a>.</p></div>
