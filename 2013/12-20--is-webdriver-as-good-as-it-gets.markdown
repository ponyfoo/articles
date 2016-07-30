<h1>Is WebDriver as good as it gets?</h1>

<p><kbd>integration-testing</kbd> <kbd>selenium</kbd> <kbd>webdriver</kbd> <kbd>grunt</kbd> <kbd>rant</kbd></p>

<blockquote><p>This blog post is part <em>rant</em>, part <em>learning experience</em>, and part <em>solutions and conclusions</em> I&#x2019;ve arrived at, while struggling with <a href="http://docs.seleniumhq.org/projects/webdriver/" target="_blank">WebDriver</a> implementations in Node.</p></blockquote>

<div><p>This blog post is part <em>rant</em>, part <em>learning experience</em>, and part <em>solutions and conclusions</em> I&#x2019;ve arrived at, while struggling with <a href="http://docs.seleniumhq.org/projects/webdriver/" target="_blank">WebDriver</a> implementations in Node.</p></div>

<div></div>

<div><p><a href="http://en.wikipedia.org/wiki/Integration_testing" target="_blank">Integration Testing</a> is a must if we&#x2019;re to build a reasonably resilient web application. It helps us figure out evident errors in the &#x201C;happy path&#x201D; of execution. However, when it comes to automated tools that enable us to do integration testing <strong>on real browsers</strong>, <em>the options available to us are pretty underwhelming</em>.</p> <p>WebDriver was <a href="http://google-opensource.blogspot.com.ar/2009/05/introducing-webdriver.html" target="_blank">introduced by Google in 2009</a>.</p> <blockquote> <p>WebDriver is a clean, fast framework for automated testing of webapps.</p> </blockquote> <p>To my knowledge, <em>documentation for WebDriver is scarce</em>, at best. Attempts to interact with it are going to be painful for you, particularly if you&#x2019;re attempting to use one of the even less documented implementations, such as the <a href="https://github.com/admc/wd" target="_blank">wd</a> package, built to run these tests using Node. The worst of it is that there <strong>doesn&#x2019;t seem to be a better alternative</strong> if you want to test with real browsers, like Chrome <em>(which you should)</em>. This is paired with the popularity of partially-implemented libraries which <em>&#x201C;sort of do what you want&#x201D;</em>, but aren&#x2019;t able to do really basic stuff like handling file uploads.</p> <p>I wish I had the time to invest effort in a Kickstarter project to improve the current state of affairs. I&#x2019;d love to see a better integration testing solution which can be implemented in any language through an API like Selenium does, and supports any browser, <strong>which just works</strong>, and whose consuming libraries were a lot better (API wise), and much better documented as well. Good documentation is <em>vastly underestimated</em> nowadays.</p> <p><a href="http://docs.seleniumhq.org/projects/webdriver/" target="_blank"><img src="https://i.imgur.com/T5uFEMC.png" alt="selenium.png"></a></p></div>

<div><h3 id="a-safety-net">A Safety Net</h3> <p>I was to automate a testing process we were doing, where we basically had a checklist of items that needed to be validated, before we could sign off on a deployment for production. The list looked sort of like this:</p> <ol> <li>Log in</li> <li>Create a project and edit it</li> <li>Upload a file</li> <li>Create a view using the uploaded file</li> <li>Validate a thumbnail is generated</li> <li>Log into mobile application</li> <li>Preview the new project</li> <li>Delete the project</li> <li>Log out</li> </ol> <p>This kind of testing helps us make sure we don&#x2019;t deploy silly mistakes to production. At a bare minimum, which is what this checklist represents, <strong>frequently used features should just work</strong>. Testing, in all its flavors, amounts to nothing if it&#x2019;s not automated. After manually going through the checklist for a couple of weeks, it was clear we needed to start automating it.</p> <p><a href="https://ponyfoo.com/2013/03/28/pragmatic-unit-testing-in-javascript" aria-label="Pragmatic Unit Testing in JavaScript">Unit Tests</a> are, of course, necessary to catch more subtle issues. Integration however aims for gross oversights, and generally being able to actually execute the application.</p> <p>I gave <a href="https://github.com/admc/wd" target="_blank" aria-label="admc/wd on GitHub">wd</a> a shot, paired with <a href="https://github.com/jmreidy/grunt-mocha-webdriver" target="_blank" aria-label="jmreidy/grunt-mocha-webdriver on GitHub">grunt-mocha-webdriver</a> so that we could automate it using <a href="http://gruntjs.com/" target="_blank" aria-label="Grunt: The JavaScript Task Runner">Grunt</a>.</p> <h3 id="grunt-automation">Grunt Automation</h3> <p>First off, we need to install the grunt task.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm install --save-dev grunt-mocha-webdriver
</code></pre> <p>Then, setting up the Grunt task was relatively easy.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">mochaWebdriver: {
  options: {
    timeout: <span class="md-code-number">1000</span> * <span class="md-code-number">60</span>,
    reporter: <span class="md-code-string">&apos;spec&apos;</span>
  },
  integration: {
    src: [<span class="md-code-string">&apos;test/integration/**/*.js&apos;</span>],
    options: {
      usePhantom: <span class="md-code-literal">true</span>,
      usePromises: <span class="md-code-literal">true</span>
    }
  }
}
</code></pre> <p>A couple of things to mention, in retrospect. At first we thought using Phantom, instead of a real browser, would be <em>such a good idea</em>, because it&#x2019;d be faster to set up, and what not. A co-worker convinced me to use promises, and I went with it. The task could definitely use a better name, but at least it worked. So I had something to get me going.</p> <p>There was one issue, though. I had to fire up the Node application myself, which meant extra work every time. I don&#x2019;t like extra work, so I wrote a task that would start a Node process, run the tests, and then stop that process. I decided that I needed to wait on the application to start listening for requests, rather than shoving them to its face. I used <a href="https://github.com/bevacqua/process-finder" target="_blank" aria-label="bevacqua/process-finder on GitHub">a little port watching module</a> which notifies me when an application starts listening on a given port, which worked just fine.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> app;

grunt.registerTask(<span class="md-code-string">&apos;integration-test-runner:start&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> done = <span class="md-code-keyword">this</span>.async();
  <span class="md-code-keyword">var</span> _ = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;lodash&apos;</span>);
  <span class="md-code-keyword">var</span> spawn = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;child_process&apos;</span>).spawn;
  <span class="md-code-keyword">var</span> finder = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;process-finder&apos;</span>);
  <span class="md-code-keyword">var</span> port = process.env.TEST_PORT || <span class="md-code-number">3333</span>;
  <span class="md-code-keyword">var</span> watcher = finder.watch({ port: port, frequency: <span class="md-code-number">400</span> });
  <span class="md-code-keyword">var</span> env = _.clone(process.env);

  <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Spawning node process to listen on port %s...&apos;</span>, port);

  env.PORT = port;
  app = spawn(<span class="md-code-string">&apos;node&apos;</span>, [<span class="md-code-string">&apos;app&apos;</span>], { stdio: <span class="md-code-string">&apos;inherit&apos;</span>, env: env });

  process.on(<span class="md-code-string">&apos;exit&apos;</span>, close);

  watcher.on(<span class="md-code-string">&apos;listen&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(pid)</span> </span>{
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Process %s listening on port %s...\nRunning tests.&apos;</span>, pid, port)

    grunt.task.run(
      <span class="md-code-string">&apos;mochaWebdriver:integration&apos;</span>,
      <span class="md-code-string">&apos;integration-test-runner:cleanup&apos;</span>
    );
    done();
  });
});

grunt.registerTask(<span class="md-code-string">&apos;integration-test-runner:cleanup&apos;</span>, close);

<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-title">close</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">if</span> (app) {
    app.kill(<span class="md-code-string">&apos;SIGHUP&apos;</span>);
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Process %s shutting down&apos;</span>, app.pid);
  } <span class="md-code-keyword">else</span> {
    <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Process not found&apos;</span>);
  }
}
</code></pre> <p>Note that <code class="md-code md-code-inline">grunt.task.run</code> won&#x2019;t actually run the task in place, but <em>enqueue</em> it so that it runs <em>after</em> the currently executing task, which is why <code class="md-code md-code-inline">done</code> is called immediately afterwards. Giving the app an uncommon port was useful because I could run the application in port <code class="md-code md-code-inline">3000</code>, like it does by default, side-by-side with the automated tests. Lastly, the cleanup logic helped me avoid issues with the Node process taking over the port, preventing those infamous <code class="md-code md-code-inline">EADDRINUSE</code> errors.</p> <h3 id="the-first-test">The First Test</h3> <p>Writing the first test was kind of awkward because I wasn&#x2019;t really familiar with promises <a href="http://www.html5rocks.com/en/tutorials/es6/promises/" target="_blank" aria-label="JavaScript Promises: There and back again">(did you know they&#x2019;re coming to JavaScript in Harmony?)</a>, but I decided to give them a shot, anyway. Here is the first test, trying out the login logic.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> port = process.env.TEST_PORT || <span class="md-code-number">3333</span>;
<span class="md-code-keyword">var</span> base = <span class="md-code-string">&apos;http://localhost:&apos;</span> + port;

it(<span class="md-code-string">&quot;handles an invalid login&quot;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(done)</span> </span>{
  <span class="md-code-keyword">this</span>.browser
    .get(base + <span class="md-code-string">&apos;/admin&apos;</span>)

    <span class="md-code-comment">// assert we&apos;re taken to the login page</span>
    .url().then(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(url)</span> </span>{
      <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Logging in...&apos;</span>);
      <span class="md-code-keyword">return</span> assert.equal(url, base + <span class="md-code-string">&apos;/login&apos;</span>);
    })

    <span class="md-code-comment">// enter invalid credentials and submit</span>
    .elementById(<span class="md-code-string">&apos;email&apos;</span>).type(<span class="md-code-string">&apos;me@here.com&apos;</span>)
    .elementById(<span class="md-code-string">&apos;pass&apos;</span>).type(<span class="md-code-string">&apos;wayoff&apos;</span>)
    .elementByTagName(<span class="md-code-string">&apos;form&apos;</span>).submit();

    <span class="md-code-comment">// assert we&apos;re still in the login page</span>
    .url().then(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(url)</span> </span>{
      <span class="md-code-keyword">return</span> assert.equal(url, base + <span class="md-code-string">&apos;/login&apos;</span>);
    })

    .then(done, done);
});
</code></pre> <p>Looks really simple, innocent, and straightforward, right? <em>I know!</em> Thing is that actually getting it to work took a lot of effort, because the API is so underdocumented, I actually had to log <code class="md-code md-code-inline">Object.keys(this.browser)</code> and go through the methods, trying to figure out which one did what I intended to do (submit the form, or type into an input). These are all symptoms of a lousy documentation. The API <em>could be worse</em>, as it at least attempts to mirror some of the methods <a href="https://ponyfoo.com/2013/06/10/uncovering-the-native-dom-api" aria-label="Uncovering the Native DOM API">found in the native DOM</a>.</p> <h3 id="context-barrier">Context Barrier</h3> <p>Suppose you get past this initial barrier. Then there&#x2019;s the context issues, and understanding how to transverse the DOM properly. If you want to get an element at random, navigate away, and then come back and select the same element, you&#x2019;re gonna have a bad time.</p> <p>This one, however, is mostly a matter of befriending promises and accumulating experience with the <code class="md-code md-code-inline">wd</code> API. In particular, the way <code class="md-code md-code-inline">.then</code> statements work is pretty cryptic. If they return an element, then that&#x2019;ll be the context for the next promise in the chain, if they don&#x2019;t then the <code class="md-code md-code-inline">browser</code> object is used, and if you prepend <code class="md-code md-code-inline">&apos;&gt;&apos;</code> to selector requests, the context is used to restrict the query. For example:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">this</span>.browser
  .elementByCssSelector(<span class="md-code-string">&apos;.some-thing&apos;</span>)
  .elementByCssSelector(<span class="md-code-string">&apos;&gt;&apos;</span>, <span class="md-code-string">&apos;.some-thing-child&apos;</span>)
  .then(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(element)</span> </span>{
    <span class="md-code-comment">// return the element or you&apos;re back to the browser</span>
    <span class="md-code-keyword">return</span> element;
  });
</code></pre> <p>Obviously, though, when you go for something like <code class="md-code md-code-inline">.text().then(function (value) {})</code>, you&#x2019;re pretty much screwed because you don&#x2019;t have a reference to the element anymore, unless you&#x2019;ve previously persisted it to a variable.</p> <h3 id="capturing-page-load-errors">Capturing Page Load Errors</h3> <p>You&#x2019;d think capturing page load errors is something that anyone would like to use. That is, logging errors that happen right after a request completes, during interpretation or execution of a piece of code. Well, if you google around, the only real solution to this problem is using client-side JavaScript, and patching <a href="https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers.onerror" target="_blank" aria-label="GlobalEventHandlers.onerror on MDN">onerror</a> so that you can keep track of errors.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">window</span>.__wd_errors = [];
<span class="md-code-built_in">window</span>.onerror = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(message, url, ln)</span> </span>{
  <span class="md-code-built_in">window</span>.__wd_errors.push({
    message: message,
    url: url,
    ln: ln
  });
};
</code></pre> <p>As WebDriver doesn&#x2019;t really provide a mechanism to inject into the response stream, or manipulate it in any way <em>(that I could find, anyways)</em>, you&#x2019;re stuck with patching the application if a <code class="md-code md-code-inline">TEST_INTEGRATION</code> environment variable or similar is turned on. On the testing side of things, you could augment the promise chain prototype to print the list of errors, after navigating to a page.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">wd.PromiseChainWebdriver.prototype.throwIfJsErrors = <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> <span class="md-code-keyword">this</span>
    .safeEval(<span class="md-code-string">&apos;window.__wd_errors&apos;</span>)
    .then(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(errors)</span> </span>{
      <span class="md-code-keyword">if</span> (errors &amp;&amp; errors.length) {
        <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;Detected %s Error(s)&apos;</span>, errors.length);

        errors.forEach(<span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(error)</span> </span>{
          <span class="md-code-built_in">console</span>.log(<span class="md-code-string">&apos;%s\nAt %s, File %s&apos;</span>, error.message, error.ln, error.url);
        });
      }
    });
};
</code></pre> <p>This helped me catch an unexpected issue. Phantom doesn&#x2019;t have <code class="md-code md-code-inline">Function.prototype.bind</code>, and <a href="https://github.com/ariya/phantomjs/issues/10522" target="_blank" aria-label="Function.prototype.bind is undefined">it won&#x2019;t get included until <code class="md-code md-code-inline">2.0.0</code></a>, which <em>doesn&#x2019;t seem to be happenning any time soon</em>. Temporarily, I added a polyfill for <code class="md-code md-code-inline">Function.prototype.bind</code> to the file which had the the page load error capturing.</p> <h3 id="file-uploading-is-a-nightmare">File Uploading <em>is a Nightmare</em></h3> <p>The worst offender of all, was file uploading. Of course, <em>documentation would&#x2019;ve helped</em>. It&#x2019;s like nobody wants to talk about integration testing anyways, so googling around doesn&#x2019;t do you a lot of good either. The best I could come up with was some information on <a href="https://github.com/admc/wd/issues/107" target="_blank" aria-label="Add file upload support">a discussion on an issue on GitHub</a>, and maybe questions about the WebDriver implementation in Java, which <em>I attempted to mirror</em>.</p> <p>I tried everything. At first, I went with the API: find the element, then use <code class="md-code md-code-inline">.sendKeys(&lt;path&gt;)</code>. Nope, that won&#x2019;t work. Okay, maybe it was just <code class="md-code md-code-inline">.type(&lt;path&gt;)</code>? No. Need a <code class="md-code md-code-inline">.click()</code> in between? Wrong again. You see, the lack of documentation makes it very hard for the consumer to know exactly how wrong their approach is. That represents a huge problem, because <strong>you&#x2019;ll keep on trying things out blindly</strong>, hoping to eventually get it right. <strong>You won&#x2019;t.</strong></p> <p>After lots of googling and a some desperate attempts, I stumbled upon a corner of the internet where I read that this functionality <a href="https://github.com/admc/wd/issues/107" target="_blank" aria-label="Add file upload support">wasn&#x2019;t implemented a few months ago</a>, sure <a href="https://github.com/admc/wd/pull/122" target="_blank" aria-label="Upload local file to remote server support">this pull request</a> sounds like it should be working, but <a href="https://github.com/detro/ghostdriver/issues/155" target="_blank" aria-label="Problem with file upload test">this issue on GhostDriver suggests otherwise</a>, and I got tired of sifting through issue lists figuring out whether what I was trying to do was even supported.</p> <p>Okay, great, so I decided to try something else. I know! I&#x2019;m fine not testing the button click itself, <strong>that&#x2019;s something a unit test could do</strong>. I care about the grand scheme of things. I <em>need to upload the file</em>, though. There&#x2019;s no getting around that. Luckily, the page I was testing wasn&#x2019;t submitting the form directly. It creates a <a href="https://developer.mozilla.org/en-US/docs/Web/API/FormData" target="_blank" aria-label="FormData in XMLHttpRequest 2 on MDN">FormData</a> object, and places the files there, and then it sends that. All I need is to <code class="md-code md-code-inline">eval</code> the right string, and it&#x2019;ll all be over!</p> <p>Some googling gave me the formula. Rather than give the file path to WebDriver, I had to hand over the <code class="md-code md-code-inline">Blob</code> data directly to the browser. Converting the file to the correct format took a little trial and error, but the renewed spirit was there. Here&#x2019;s a small addition to the <code class="md-code md-code-inline">wd</code> promise chain prototype, so I could re-use my awesome file upload hack, some day.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">wd.PromiseChainWebdriver.prototype.uploadSomething = function (file, scope) {
  console.log(&apos;Attempting file upload...&apos;);

  var mime = require(&apos;mime&apos;);
  var name = path.basename(file);
  var blob = fs.readFileSync(file, { encoding: &apos;base64&apos; });
  var code = [
    util.format(&apos;var bytes = atob(&quot;%s&quot;);&apos;, blob),
    &apos;var codepoints = Array.prototype.map.call(bytes, function (n) { return n.charCodeAt(0); });&apos;,
    &apos;var data = new Uint8Array(codepoints);&quot;,
    util.format(&apos;var blob = new Blob([data], { type: &quot;%s&quot; });&apos;, mime.lookup(file)),
    util.format(&apos;blob.name = &apos;%s&apos;;&apos;, name),
    &apos;var formData = new FormData();&apos;,
    util.format(&apos;formData.append(&quot;%s&quot;, blob);&apos;
  ].join(&apos; &apos;);

  return this
    .safeEval(code)
    .then(function () {
      console.log(&apos;File upload in progress.&apos;);
    });
};
</code></pre> <p>Feeling <em>great!</em> Let&#x2019;s do this! &#x2026;nope, <em>not working</em>. <a href="https://github.com/ariya/phantomjs/issues/11013" target="_blank" aria-label="new Blob() throws error">Blob is unsupported in Phantom &lt; <code class="md-code md-code-inline">2.0.0</code></a>, just like <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind" target="_blank" aria-label="Function.prototype.bind on MDN"><code class="md-code md-code-inline">Function.prototype.bind</code></a>. The <a href="https://github.com/eligrey/Blob.js" target="_blank" aria-label="eligrey/Blob.js on GitHub">polyfill</a> I tried out didn&#x2019;t work either, and I simply moved on to Chrome.</p> <h3 id="moving-to-chrome">Moving to Chrome</h3> <p>Okay, fine. Rather than using the unreliable Ghost browser, I needed the real thing, and so I went with Chrome. To run tests with Chrome, I needed to change things up quite a bit. First off, I found a <a href="https://github.com/vvo/selenium-standalone" target="_blank" aria-label="vvo/selenium-standalone on GitHub">really simple selenium server installer</a> which did the trick of firing up a Selenium server for me.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm install -g selenium-standalone
</code></pre> <p>Now I could fire up an instance in my command line. <em>By the way, it requires Java!</em></p> <pre class="md-code-block"><code class="md-code md-lang-bash">start-selenium
</code></pre> <p>Wait a minute&#x2026; that&#x2019;s too easy! Oh yeah, that&#x2019;s right, <code class="md-code md-code-inline">grunt-mocha-webdriver</code> doesn&#x2019;t run against local selenium instances, even though <a href="https://github.com/jmreidy/grunt-mocha-webdriver/pull/18" target="_blank" aria-label="Run against local selenium instances">a pull request</a>, which adds that functionality, <strong>is already a month old</strong>. I went ahead and <a href="https://github.com/bevacqua/grunt-mocha-webdriver-painful" target="_blank" aria-label="bevacqua/grunt-mocha-webdriver-painful on GitHub">created a package</a> out of the pull request, using that I could test against the local selenium instance created by <code class="md-code md-code-inline">start-selenium</code>.</p> <p>I really wanted to keep the selenium instance contained in the Grunt task, as well, so I went ahead and <a href="https://github.com/bevacqua/selenium-standalone-painful" target="_blank" aria-label="bevacqua/selenium-standalone-painful on GitHub">cloned <code class="md-code md-code-inline">selenium-standalone</code></a>, adding a programmatic API. After that, it was just a matter of pulling it into a new Grunt task. That task would spawn a selenium server instance consuming the API I&#x2019;ve just written.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> selenium = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;selenium-standalone-painful&apos;</span>).start({
  stdio: <span class="md-code-string">&apos;pipe&apos;</span>
});

<span class="md-code-keyword">var</span> ready = <span class="md-code-regexp">/Started org\.openqa\.jetty\.jetty\.Server/i</span>;

selenium.stdout.on(<span class="md-code-string">&apos;data&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">if</span> (ready.test(data.toString())) {
    grunt.task.run(<span class="md-code-string">&apos;the-next-one&apos;</span>);
  }
});

process.on(<span class="md-code-string">&apos;exit&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">if</span> (selenium) {
    selenium.kill(<span class="md-code-string">&apos;SIGHUP&apos;</span>);
  }
});
</code></pre> <p>That&#x2019;s it! Then I decided to improve the reusability by pulling it out of its host project.</p> <h3 id="introducing-grunt-integration">Introducing <code class="md-code md-code-inline">grunt-integration</code></h3> <p>I built a tool specifically to deal with the <del>issues</del> <ins>ordeal</ins> I went through while ramping up on my integration testing experience on Node applications. Concretely, these are the features I want in an integration testing module, and also the goals of <a href="https://github.com/bevacqua/grunt-integration" target="_blank" aria-label="bevacqua/grunt-integration on GitHub"><code class="md-code md-code-inline">grunt-integration</code></a>:</p> <ul> <li>Start a local selenium server instance</li> <li>Start a local program, such as <code class="md-code md-code-inline">node</code> application</li> <li>Wait for the program to listen on a specific port</li> <li>Execute integration tests using <a href="https://github.com/visionmedia/mocha" target="_blank" aria-label="visionmedia/mocha on GitHub">Mocha</a> and <a href="https://github.com/admc/wd" target="_blank" aria-label="admc/wd on GitHub">WebDriver</a></li> <li>Using real browsers, such as Chrome</li> <li>Automatically, in one command</li> <li>Less painful installation process, please!</li> </ul> <p>I&#x2019;m considering adding some extensions to <code class="md-code md-code-inline">wd</code> so that dealing with some of the issues I&#x2019;ve described here is not so painful. The <code class="md-code md-code-inline">wd</code> API could definitely get some love, but you can&#x2019;t do a lot better than what it currently has. HTTP injection would be something that I&#x2019;d love to see there, but I don&#x2019;t think its even possible with Selenium.</p> <p>You can <a href="https://github.com/bevacqua/grunt-integration" target="_blank" aria-label="bevacqua/grunt-integration on GitHub">check out the ongoing progress on GitHub</a>, and maybe even collaborate with your expertise!</p></div>
