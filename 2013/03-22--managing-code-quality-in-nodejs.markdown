<h1>Managing Code Quality in NodeJS</h1>

<blockquote><p>I&#x2019;ve mentioned <a href="https://ponyfoo.com/2013/01/18/continuous-integration-and-automated-deployments">CI</a> and <a href="https://ponyfoo.com/2013/01/18/asset-management-in-node">static asset management</a> in the past. Now I want to talk about code quality.</p><p>This article is mostly a follow up on the <a href="https://ponyfoo.com/2013/01/18/continuous-integration-and-automated-deployments">CI</a> post. I&#x2019;ll &#x2026;</p></blockquote>

<div><kbd>build</kbd> <kbd>nodejs</kbd> <kbd>grunt</kbd> <kbd>assetify</kbd> <kbd>unit-testing</kbd></div>

<div><p>I&#x2019;ve mentioned <a href="https://ponyfoo.com/2013/01/18/continuous-integration-and-automated-deployments">CI</a> and <a href="https://ponyfoo.com/2013/01/18/asset-management-in-node">static asset management</a> in the past. Now I want to talk about code quality.</p></div>

<div></div>

<div><p>This article is mostly a follow up on the <a href="https://ponyfoo.com/2013/01/18/continuous-integration-and-automated-deployments">CI</a> post. I&#x2019;ll describe how <a href="https://ponyfoo.com/gruntjs.com">Grunt</a> helped me change the <em>test</em> and <em>build</em> processes used in this blog&#x2019;s <a href="https://github.com/bevacqua/ponyfoo" target="_blank">engine</a>.</p> <blockquote> <p>Before using Grunt, I didn&#x2019;t really have a <strong>real</strong> build process. <em>Sure</em>, <code class="md-code md-code-inline">git push heroku master</code> <em>triggered a build</em> on their end, but I didn&#x2019;t control any of it, all I did was <code class="md-code md-code-inline">node app.js</code>.</p> </blockquote> <p>Similarly, my <a href="https://travis-ci.org/bevacqua/ponyfoo/builds" target="_blank">Travis-CI hook</a> just made sure there weren&#x2019;t any conflicts with my npm packages. I could do <em>better</em>.</p></div>

<div><p>I&#x2019;ve started leaning towards making the engine more <em>testable</em>, I figured I had to start <strong>somewhere</strong>. For me, that first step was to <em>use a task runner</em>.</p> <h1 id="grunt-gruntjscom-grunt-the-javascript-task-runner-proper-build-process"><a href="https://ponyfoo.com/gruntjs.com" aria-label="Grunt: The JavaScript Task Runner">Grunt</a>: proper Build Process</h1> <p>After spending a few days toying with Grunt, I must say it&#x2019;s <em>really exciting</em> to use a build tool that <em>just works</em>, rather than get in your way. I learned a couple of interesting things, and even created my very own <a href="https://github.com/bevacqua/grunt-assetify" target="_blank" aria-label="grunt-assetify on GitHub">grunt plugin for assetify</a>.</p> <p>Grunt runs from the command line, and you should install it globally, using <code class="md-code md-code-inline">npm install -g grunt-cli</code>. Grunt will look for <code class="md-code md-code-inline">Gruntfile.js</code>, and run the function exported by that module, passing a helper as an argument.</p> <p>Here&#x2019;s a sample listing that does basically the same as the traditional <code class="md-code md-code-inline">node app.js</code> command used to. Save this as <code class="md-code md-code-inline">Gruntfile.js</code>:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-pi">&apos;use strict&apos;</span>;

<span class="md-code-built_in">module</span>.exports = <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">(grunt)</span> </span>{
    grunt.initConfig({});
    grunt.registerTask(<span class="md-code-string">&apos;app&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span><span class="md-code-params">()</span></span>{
        <span class="md-code-keyword">var</span> done = <span class="md-code-keyword">this</span>.async(),
            app = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./app.js&apos;</span>);

        app.start(done);
    });
};
</code></pre> <p>You should make a very minor change to your <code class="md-code md-code-inline">app.js</code>, and add <code class="md-code md-code-inline">app.on(&apos;close&apos;, done);</code>. This will signal grunt to shut down when the server is closed, rather than as soon as it&#x2019;s idle.</p> <p>Now you can <code class="md-code md-code-inline">grunt app</code> to run your web application using <code class="md-code md-code-inline">grunt</code>. How cool is that? <em>Fine</em>, it&#x2019;s not very amusing.</p> <p>The amusing part is the kind of things Grunt <em>enables</em> you to do, now that it&#x2019;s set up. You can <em>integrate</em> other stuff you might&#x2019;ve shied away from because it was too cumbersome to run, or complicated to configure. Grunt solves some of that for you.</p> <p><img alt="grunt-cli-sample.png" title="typical grunt console output" class="" src="https://i.imgur.com/i28vdBO.png"></p> <p>I&#x2019;ll now go over some of the tools I integrated into my <em>build process</em> since I switched to Grunt. I&#x2019;m not saying Grunt enabled me to use these tools, it just <em>made my life easier</em>.</p> <h2 id="lint-static-code-quality-analysis">Lint: Static Code Quality Analysis</h2> <p><a href="http://www.jshint.com/" target="_blank" aria-label="JSHint">JSHint</a> is a In case you don&#x2019;t know it, <a href="http://jslint.com/" target="_blank" aria-label="JSLint by Douglas Crockford">JSLint</a> helps catch syntax errors (and <em>coding style issues</em>) in your code. JSHint is highly configurable fork of JSLint, and their <a href="http://www.jshint.com/docs/" target="_blank" aria-label="JSHint Documentation">documentation</a> explains each option in detail.</p> <p>You should consider JSHint as your very first unit test. Since JavaScript doesn&#x2019;t have a formal compiler, a linting tool such as JSHint will have to do.</p> <p>You can find the GitHub repository to their Grunt plugin <a href="https://github.com/gruntjs/grunt-contrib-jshint" target="_blank" aria-label="JSHint plugin for Grunt">here</a>, and the actual sources of JSHint are <a href="https://github.com/jshint/jshint" target="_blank" aria-label="JSHint on GitHub">here</a>.</p> <p>Here&#x2019;s how I added it to my <code class="md-code md-code-inline">Gruntfile.js</code> (displaying only the relevant code):</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">grunt.initConfig({
    jshint: {
        node: {
            files: {
                src: [
                    <span class="md-code-string">&apos;gruntfile.js&apos;</span>,
                    <span class="md-code-string">&apos;src/**/*.js&apos;</span>,
                    <span class="md-code-string">&apos;!src/static/**/*.js&apos;</span>,
                    <span class="md-code-string">&apos;test/spec/**/*.js&apos;</span>
                ]
            },
            options: {
                jshintrc: <span class="md-code-string">&apos;.jshintrc&apos;</span>
            }
        },
        browser: {
            files: {
                src: [
                    <span class="md-code-string">&apos;src/static/**/*.js&apos;</span>,
                    <span class="md-code-string">&apos;!src/static/config/*.js&apos;</span>,
                    <span class="md-code-string">&apos;!src/static/.bin/**/*.js&apos;</span>,
                    <span class="md-code-string">&apos;!src/static/js/vendor/**/*.js&apos;</span>
                ]
            },
            options: {
                jshintrc: <span class="md-code-string">&apos;.jshintrc-browser&apos;</span>
            }
        }
    }
});

grunt.loadNpmTasks(<span class="md-code-string">&apos;grunt-contrib-jshint&apos;</span>);
</code></pre> <p>This brings us to an interesting point about Grunt, which is <em>targets</em>. The <code class="md-code md-code-inline">node</code> and <code class="md-code md-code-inline">browser</code> properties are just <em>targets</em>, which means that <code class="md-code md-code-inline">grunt jshint:node</code> will run JSHint with the <em>first set</em> of options, and <code class="md-code md-code-inline">grunt jshint:browser</code> will run the <em>second set</em>. <code class="md-code md-code-inline">grunt jshint</code> will do <em>both</em> in turn.</p> <p>If you&#x2019;re not <em>already</em> linting your code, it will take you a few moments to make it comply with a default JSHint configuration, but it will be <em>worth it</em>.</p> <h2 id="unit-testing">Unit Testing</h2> <p>If you&#x2019;re looking to get started with Unit Testing in NodeJS I&#x2019;ll just recommend you to read <a href="http://tddjs.com/" target="_blank" aria-label="Test-Driven Development in JavaScript">TDD.JS</a>.</p> <p>I&#x2019;ll expand on this topic on a later post, since there&#x2019;s a lot to write on the topic, particularly on testing a dynamically typed language such as JavaScript, which in my opinion is both harder but more <strong>enjoyable</strong> (when done right).</p> <p>Meanwhile, you <em>must</em> look at <a href="http://superherojs.com/" target="_blank">Superhero.js</a>, which offers a bunch of JavaScript articles, videos, and books you should definitely keep an eye out for!</p> <h2 id="again-with-static-asset-management">Again with Static Asset Management</h2> <p>I&#x2019;ve mentioned in the past how I was building <a href="https://ponyfoo.com/2013/01/18/asset-management-in-node" aria-label="Asset management in Node">assetify</a>. It hasn&#x2019;t changed much lately, except for a fingerprinting (as in <code class="md-code md-code-inline">/all.js?v=2r4c-19by10z</code>) middleware that allows us to set far-future <code class="md-code md-code-inline">Expires</code> headers. Other than that, I did add a <a href="https://github.com/bevacqua/grunt-assetify" target="_blank" aria-label="grunt plugin for assetify">grunt plugin</a> for assetify.</p> <p>If you&#x2019;re interested in plugging assetify into your Grunt configuration, you should definitely check out the <a href="https://github.com/bevacqua/grunt-assetify-example" target="_blank" aria-label="grunt-assetify example usage">example repository</a> I&#x2019;ve set up on GitHub. The Grunt setup on that repository also has JSHint and Jasmine tasks, so it might even be a good starting point for configuring a nice <code class="md-code md-code-inline">Gruntfile.js</code></p> <p>Lastly, <em>remember</em>:</p> <blockquote> <p>Stand watchful, and continuously assess the quality of your code. These guidelines may make maintaining, refactoring, and developing your application leaner, more straightforward, and <em>less bug-prone</em>. You are the one who&#x2019;s going to have to stand by them. Incorporate them as a part of your everyday work, and you&#x2019;ll soon start to <em>reap the benefits</em>.</p> </blockquote> <p><img alt="continuous.gif" title="Stand watchful" class="" src="https://i.imgur.com/Pzfnf7z.gif"></p></div>
