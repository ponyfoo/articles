<div><blockquote>
  <h1>My First Gulp Adventure</h1>
  <div><p>I decided to take a gulp of Gulp and use it in one of my latest projects, to help me with releases. I wrote a Gulpfile, which lets me write some code to define the tasks &#x2026;</p></div>
</blockquote></div>

<div><p>I decided to take a gulp of Gulp and use it in one of my latest projects, to help me with releases. I wrote a Gulpfile, which lets me write some code to define the tasks enumerated below.</p></div>

<div></div>

<div><ul> <li>Lint my source code</li> <li>Run unit tests</li> <li>Clean my distribution directory</li> <li>Build the distribution files, minified and otherwise</li> <li>Get the file size of both the regular and minified versions</li> <li>Bump the package version for <code class="md-code md-code-inline">npm</code> and <code class="md-code md-code-inline">bower</code></li> <li>Push a new tag to <code class="md-code md-code-inline">git</code>, to update the Bower version</li> <li>Publish the updated version to <code class="md-code md-code-inline">npm</code></li> </ul> <p>In this article I aim to explain what I did, how I did it, and the reasons why I made some of the choices that I did. The only real problem I had had to do with synchronicity. I felt it would be interesting walking you through the process. It may help you get started with Gulp!</p> <p><img src="https://i.imgur.com/ApIcjlI.png" alt="rocket.png" title="The Gulp Rocket!"></p></div>

<div><h1 id="gulp-s-dependency-system">Gulp&#x2019;s Dependency System</h1> <p>It seems that <a href="https://github.com/gulpjs/gulp/issues/96" target="_blank" aria-label="&apos;Support running task synchronously&apos; - issue on GitHub">they haven&#x2019;t yet settled</a> on an approach to tasks depending on other tasks Some people write their own versions of how this should look like, while the author suggests we use the &#x201C;dependency system&#x201D;. Running things in parallel sounds appealing on paper, but it loses value as you realize you probably want to log output serially, in order to be able to make sense out of it. A tempting possibility might be buffering the output produced by each task, and then flushing it as each task finishes, but this feels like too convoluted to ever be elegant, or a thing.</p> <p>By default, Gulp tasks run asynchronously, and the &#x201C;dependency system&#x201D;, allows you to fire pre-requisite tasks before a given task is able to run. <strong>It feels verbose and unlike the rest of Gulp</strong>.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;one&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(cb)</span> </span>{
  cb();
});

gulp.task(<span class="md-code-string">&apos;two&apos;</span>, [<span class="md-code-string">&apos;one&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(cb)</span> </span>{
  cb();
});

gulp.task(<span class="md-code-string">&apos;three&apos;</span>, [<span class="md-code-string">&apos;two&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(cb)</span> </span>{
  cb();
});

gulp.task(<span class="md-code-string">&apos;default&apos;</span>, [<span class="md-code-string">&apos;three&apos;</span>]);
</code></pre> <p>If you know of <em>a better way</em>, within Gulp, please let me know! I imagine running tasks in series is a pretty darn common thing, and I don&#x2019;t understand the reason why they made the task runner work asynchronously in the first place.</p> <p>That being said, the rest of the <code class="md-code md-code-inline">&apos;use gulp&apos;;</code> experience was pretty awesome. Let&#x2019;s delve into that!</p> <h1 id="getting-started">Getting Started</h1> <p>First things first, you&#x2019;ll need to install Gulp both globally, just one time; and locally, for your package.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm i -g gulp
npm i -D gulp
</code></pre> <p>Next, you create a <code class="md-code md-code-inline">gulpfile.js</code> file which looks like below.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> gulp = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp&apos;</span>);

gulp.task(<span class="md-code-string">&apos;default&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
});
</code></pre> <p>Great. You have your default task, and it does a whole lot of nothing. <em>Asynchronously!</em></p> <h1 id="building-your-package">Building your package</h1> <p>Let&#x2019;s have that build and minify our code. We&#x2019;ll need to install a few <code class="md-code md-code-inline">npm</code> packages to do that.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">npm i -D gulp-uglify gulp-concat gulp-rename
</code></pre> <p>Here&#x2019;s how you would create your first task, which bundles, and minifies our source code, writing the results to disk.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> gulp = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp&apos;</span>);
<span class="md-code-keyword">var</span> concat = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-concat&apos;</span>);
<span class="md-code-keyword">var</span> rename = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-rename&apos;</span>);
<span class="md-code-keyword">var</span> uglify = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-uglify&apos;</span>);
<span class="md-code-keyword">var</span> pkg = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./package.json&apos;</span>);

gulp.task(<span class="md-code-string">&apos;build&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/*.js&apos;</span>)
    .pipe(concat(pkg.name + <span class="md-code-string">&apos;.js&apos;</span>))
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>))
    .pipe(rename(pkg.name + <span class="md-code-string">&apos;.min.js&apos;</span>))
    .pipe(uglify())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>));
});
</code></pre> <p>Let&#x2019;s go line by line to get a better feel of what we&#x2019;re doing.</p> <h3 id="some-require-statements">Some <code class="md-code md-code-inline">require</code> statements</h3> <p>Here we&#x2019;re just getting the plugin packages, nothing special. Just regular [Common.JS] <code class="md-code md-code-inline">require</code> statements. Let&#x2019;s skip those lines, since they don&#x2019;t add much.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> gulp = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp&apos;</span>);
<span class="md-code-keyword">var</span> concat = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-concat&apos;</span>);
<span class="md-code-keyword">var</span> rename = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-rename&apos;</span>);
<span class="md-code-keyword">var</span> uglify = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-uglify&apos;</span>);
</code></pre> <p>Contrary to how Grunt operated, we need to <code class="md-code md-code-inline">require(&apos;gulp&apos;)</code> in our Gulpfiles, which I think is better than exporting a function, like we do when using Grunt, as it&#x2019;s <em>just unnecessary</em>.</p> <h3 id="var-pkg-require-packagejson"><code class="md-code md-code-inline">var pkg = require(&apos;./package.json&apos;);</code></h3> <p>I&#x2019;m just leveraging <code class="md-code md-code-inline">package.json</code> metadata in order to be able to use the package name as the file name, which would allow me to copy and paste this Gulpfile into some other project, with similar requirements, and not even need to edit the build!</p> <h3 id="gulptask-build-function"><code class="md-code md-code-inline">gulp.task(&apos;build&apos;, function () {</code></h3> <p>Here I&#x2019;m defining a <code class="md-code md-code-inline">build</code> task, which can be required by other tasks as a pre-requisite, executed as a sub-task using <code class="md-code md-code-inline">gulp.run(&apos;build&apos;)</code>, or executed via the CLI using <code class="md-code md-code-inline">gulp build</code>. Since I&#x2019;m not taking a <code class="md-code md-code-inline">cb</code> argument, I should return the build stream, so that I can make the task a dependency which would finish when the stream is closed.</p> <h3 id="return-gulpsrc-src-js"><code class="md-code md-code-inline">return gulp.src(&apos;./src/*.js&apos;)</code></h3> <p>The <code class="md-code md-code-inline">return</code> statement signals that we want the task to run synchronously. In other words, the task won&#x2019;t <em>&#x201C;finish&#x201D;</em> immediately, and thus it will effectively block other tasks if we were to add it as a dependency, until the stream is closed.</p> <blockquote> <p>Note that <code class="md-code md-code-inline">gulp.run</code> has been <a href="https://github.com/gulpjs/gulp/blob/aea0f93eaae9204a0a42e8e83372266915b089b5/CHANGELOG.md#35" target="_blank" aria-label="Gulp Changes in v3.5">deprecated in Gulp <code class="md-code md-code-inline">v3.5</code></a> and will be <a href="https://github.com/gulpjs/gulp/blob/aea0f93eaae9204a0a42e8e83372266915b089b5/index.js#L16" target="_blank" aria-label="gulp.run removal in v4">removed entirely in Gulp <code class="md-code md-code-inline">v4</code></a>.</p> </blockquote> <p>The examples below would wait until our <code class="md-code md-code-inline">&apos;build&apos;</code> task is completed.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.run(<span class="md-code-string">&apos;build&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-comment">// build is complete!</span>
});
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;release&apos;</span>, [<span class="md-code-string">&apos;build&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-comment">// build is complete, release the kraken!</span>
});
</code></pre> <p>Lastly, <code class="md-code md-code-inline">gulp.src(&apos;./src/*.js&apos;)</code> tells Gulp that it&#x2019;ll have to work with the files which <a href="http://gruntjs.com/configuring-tasks#globbing-patterns" target="_blank" aria-label="Globbing Patterns Explained">match the globbing expression</a> <code class="md-code md-code-inline">&apos;./src/*.js&apos;</code>, or all JavaScript files in the <code class="md-code md-code-inline">./src</code> directory. At this point, however, Gulp doesn&#x2019;t know anything else, only that it needs to work with those source files.</p> <h3 id="pipe-concat-pkgname-js"><code class="md-code md-code-inline">.pipe(concat(pkg.name + &apos;.js&apos;))</code></h3> <p>Here things start getting interesting. First, <code class="md-code md-code-inline">concat(pkg.name + &apos;.js&apos;)</code> <a href="https://github.com/wearefractal/gulp-concat/blob/master/index.js#L43" target="_blank" aria-label="The stream returned by concat">constructs a stream</a> that will bundle together all files piped into it. Then, <code class="md-code md-code-inline">.pipe()</code> will do exactly that, pipe the source files chosen matching the globbing expression in the previous step. This results in source files getting bundled <a href="https://github.com/wearefractal/gulp-concat/blob/master/index.js#L39" target="_blank" aria-label="Data emitted by the concat stream">into a single data blob</a>.</p> <h3 id="pipe-gulpdest-dist"><code class="md-code md-code-inline">.pipe(gulp.dest(&apos;./dist&apos;))</code></h3> <p>Up until this point, everything has been done in memory. The <code class="md-code md-code-inline">gulp.dest(&apos;./dist&apos;)</code> statement returns a write stream which writes to disk, in the <code class="md-code md-code-inline">./dist</code> directory. Once the <code class="md-code md-code-inline">concat</code> operation is completed, a single data blob will be piped into the <code class="md-code md-code-inline">dest</code> stream, writing the bundle to disk.</p> <h3 id="pipe-rename-pkgname-minjs"><code class="md-code md-code-inline">.pipe(rename(pkg.name + &apos;.min.js&apos;))</code></h3> <p>Doing <code class="md-code md-code-inline">rename(pkg.name + &apos;.min.js&apos;)</code> creates a stream which <a href="https://github.com/hparra/gulp-rename/blob/master/index.js#L41" target="_blank" aria-label="Renaming the destination filename">changes the destination filename</a> from what we originally set when creating the <code class="md-code md-code-inline">concat()</code> stream. Subsequent calls to <code class="md-code md-code-inline">.dest()</code> will be told to write to this filename, instead.</p> <h3 id="pipe-uglify"><code class="md-code md-code-inline">.pipe(uglify())</code></h3> <p>You can probably guess that <code class="md-code md-code-inline">uglify()</code> creates a stream, which minifies the bundle and emits that.</p> <h3 id="pipe-gulpdest-dist"><code class="md-code md-code-inline">.pipe(gulp.dest(&apos;./dist&apos;));</code></h3> <p>Lastly, we pipe into <code class="md-code md-code-inline">./dist</code> again, writing bundled, minified code, into a single file.</p> <p>So that&#x2019;s it, let&#x2019;s look at that task again.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;build&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/*.js&apos;</span>)
    .pipe(concat(pkg.name + <span class="md-code-string">&apos;.js&apos;</span>))
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>))
    .pipe(rename(pkg.name + <span class="md-code-string">&apos;.min.js&apos;</span>))
    .pipe(uglify())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>));
});
</code></pre> <p>I&#x2019;d like to see the minified file size every time I run this task, and I could use <code class="md-code md-code-inline">gulp-size</code> to do that. Let&#x2019;s see how that works.</p> <h3 id="pipe-size"><code class="md-code md-code-inline">.pipe(size())</code></h3> <p>First, we need to install <code class="md-code md-code-inline">gulp-size</code>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm i -D gulp-size
</code></pre> <p>Then, we just add that stream to our pipeline, right after we uglify (minify) our code.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;build&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/*.js&apos;</span>)
    .pipe(concat(pkg.name + <span class="md-code-string">&apos;.js&apos;</span>))
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>))
    .pipe(rename(pkg.name + <span class="md-code-string">&apos;.min.js&apos;</span>))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>));
});
</code></pre> <p>Easy enough! Don&#x2019;t forget to <code class="md-code md-code-inline">require</code> it.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> size = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-size&apos;</span>);
</code></pre> <h1 id="pushing-a-release">Pushing a release</h1> <p>Once you&#x2019;re able to build your package, you&#x2019;d probably want to automate the tedious process of releasing a package update. For example, here&#x2019;s everything that happens when I release a new version of <a href="https://github.com/bevacqua/contra" target="_blank" aria-label="bevacqua/contra on GitHub"><code class="md-code md-code-inline">contra</code></a>.</p> <ul> <li>I build the regular and minified library versions</li> <li>I run unit tests to ensure everything is working as expected</li> <li>I bump the version number in both <code class="md-code md-code-inline">package.json</code> and <code class="md-code md-code-inline">bower.json</code></li> <li>I create a commit with those changes</li> <li>I create a tag for the release</li> <li>I push those changes so that Bower can tell I updated the library</li> <li>I publish an update to <code class="md-code md-code-inline">npm</code></li> </ul> <p>Yeah, that ain&#x2019;t gonna work if I&#x2019;d like to push several updates in short succession for any reason, or if I have to maintain any more libraries. And even if I don&#x2019;t, doing all of that by hand introduces the very likely possibility that I make a mistake, or forget one of the steps, resulting in unhappy package consumers.</p> <blockquote> <p>No, it&#x2019;s <strong>better to automate releases.</strong></p> </blockquote> <p>Imagine if I were able to just do <code class="md-code md-code-inline">gulp release</code> and have all of that happen. Actually, that&#x2019;s <em>exactly how it is set up.</em> In one task, I bump the package version, I do all the git-related operations in another, and I push to <code class="md-code md-code-inline">npm</code> in a third task. Let&#x2019;s dissect each of them.</p> <h2 id="bumping-packages">Bumping packages</h2> <p>Bumping the package version is pretty straightforward, and we can use <a href="https://github.com/stevelacy/gulp-bump" target="_blank" aria-label="stevelacy/gulp-bump on GitHub"><code class="md-code md-code-inline">gulp-bump</code></a>, which does that, and only that.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;bump&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src([<span class="md-code-string">&apos;./package.json&apos;</span>, <span class="md-code-string">&apos;./bower.json&apos;</span>])
    .pipe(bump())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./&apos;</span>));
});
</code></pre> <p>I don&#x2019;t even have to decompose that one, you just tell Gulp to read from <code class="md-code md-code-inline">package.json</code> and <code class="md-code md-code-inline">bower.json</code>, or whichever versioned JSON manifests you have, and pipe that through <code class="md-code md-code-inline">bump()</code> and into the <code class="md-code md-code-inline">dest</code> write stream. Easy peasy!</p> <p>Just remember to install and <code class="md-code md-code-inline">require</code> <a href="https://github.com/stevelacy/gulp-git" target="_blank" aria-label="stevelacy/gulp-git on GitHub">gulp-bump</a>.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm i -D gulp-bump
</code></pre> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> bump = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-bump&apos;</span>);
</code></pre> <h2 id="tagging-on-git">Tagging on <code class="md-code md-code-inline">git</code></h2> <p>There&#x2019;s an awesome <code class="md-code md-code-inline">git</code> package for Gulp in <a href="https://github.com/stevelacy/gulp-git" target="_blank" aria-label="stevelacy/gulp-git on GitHub"><code class="md-code md-code-inline">gulp-git</code></a>. It does everything. It commits, it tags, it pushes, and everything else. Seriously, <a href="https://github.com/stevelacy/gulp-git" target="_blank" aria-label="stevelacy/gulp-git on GitHub">go look at it&#x2019;s documentation</a>. Terrific! The author <a href="https://ponyfoo.com/2014/01/20/how-to-design-great-programs" aria-label="How to Design Great Programs">documented it like a gentleman</a>, good stuff.</p> <p>In this particular task, I chose to use the <code class="md-code md-code-inline">package.json</code> data again. I use it to sign my commit with the release number. Then I push the <code class="md-code md-code-inline">master</code> branch to the <code class="md-code md-code-inline">origin</code> remote, and I include the <code class="md-code md-code-inline">--tags</code>, so that I don&#x2019;t have to do that by hand either.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;tag&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> pkg = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./package.json&apos;</span>);
  <span class="md-code-keyword">var</span> v = <span class="md-code-string">&apos;v&apos;</span> + pkg.version;
  <span class="md-code-keyword">var</span> message = <span class="md-code-string">&apos;Release &apos;</span> + v;

  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./&apos;</span>)
    .pipe(git.commit(message))
    .pipe(git.tag(v, message))
    .pipe(git.push(<span class="md-code-string">&apos;origin&apos;</span>, <span class="md-code-string">&apos;master&apos;</span>, <span class="md-code-string">&apos;--tags&apos;</span>))
    .pipe(gulp.dest(<span class="md-code-string">&apos;./&apos;</span>));
});
</code></pre> <p>Being able to release just like that is pretty awesome. However, this task depends directly on the bump task in order to succeed, as it&#x2019;ll use the version number to create the tag. We&#x2019;ll get into the flow later, for now this just works if we run them like below.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">gulp bump
gulp tag
</code></pre> <p>Soon we&#x2019;ll check out how these dependencies can be sorted out.</p> <h2 id="publishing-on-npm">Publishing on <code class="md-code md-code-inline">npm</code></h2> <p>I couldn&#x2019;t find a <code class="md-code md-code-inline">gulp-npm</code> package for my <code class="md-code md-code-inline">npm publish</code> purposes, so I just created my own task, without developing a full-fledged plugin.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;npm&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(done)</span> </span>{
  <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;child_process&apos;</span>).spawn(<span class="md-code-string">&apos;npm&apos;</span>, [<span class="md-code-string">&apos;publish&apos;</span>], { stdio: <span class="md-code-string">&apos;inherit&apos;</span> })
    .on(<span class="md-code-string">&apos;close&apos;</span>, done);
});
</code></pre> <p>In case you&#x2019;ve never seen it before, setting <code class="md-code md-code-inline">{ stdio: &apos;inherit&apos; }</code> when spawning a child process, then the child will use your standard input, output, and error. In other words, <code class="md-code md-code-inline">npm publish</code> will be able to print its output on your terminal when you run <code class="md-code md-code-inline">gulp npm</code>.</p> <h3 id="putting-it-all-together">Putting it all together</h3> <p>To put it all together, all that&#x2019;s required is adding an array with the dependencies to each task. Here&#x2019;s is <a href="https://github.com/bevacqua/contra" target="_blank" aria-label="bevacqua/contra on GitHub">contra</a>&#x2019;s complete Gulpfile.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-keyword">var</span> gulp = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp&apos;</span>);
<span class="md-code-keyword">var</span> bump = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-bump&apos;</span>);
<span class="md-code-keyword">var</span> git = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-git&apos;</span>);
<span class="md-code-keyword">var</span> jshint = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-jshint&apos;</span>);
<span class="md-code-keyword">var</span> mocha = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-mocha&apos;</span>);
<span class="md-code-keyword">var</span> clean = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-clean&apos;</span>);
<span class="md-code-keyword">var</span> rename = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-rename&apos;</span>);
<span class="md-code-keyword">var</span> uglify = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-uglify&apos;</span>);
<span class="md-code-keyword">var</span> size = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;gulp-size&apos;</span>);

gulp.task(<span class="md-code-string">&apos;lint&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/*.js&apos;</span>)
    .pipe(jshint(<span class="md-code-string">&apos;.jshintrc&apos;</span>))
    .pipe(jshint.reporter(<span class="md-code-string">&apos;jshint-stylish&apos;</span>));
});

gulp.task(<span class="md-code-string">&apos;mocha&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  gulp.src(<span class="md-code-string">&apos;./test/*.js&apos;</span>)
    .pipe(mocha({ reporter: <span class="md-code-string">&apos;list&apos;</span> }));
});

gulp.task(<span class="md-code-string">&apos;clean&apos;</span>, <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./dist&apos;</span>, { read: <span class="md-code-literal">false</span> })
    .pipe(clean());
});

gulp.task(<span class="md-code-string">&apos;build&apos;</span>, [<span class="md-code-string">&apos;test&apos;</span>, <span class="md-code-string">&apos;clean&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/contra.js&apos;</span>)
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>))
    .pipe(rename(<span class="md-code-string">&apos;contra.min.js&apos;</span>))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>));
});

gulp.task(<span class="md-code-string">&apos;build-shim&apos;</span>, [<span class="md-code-string">&apos;build&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./src/contra.shim.js&apos;</span>)
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>))
    .pipe(rename(<span class="md-code-string">&apos;contra.shim.min.js&apos;</span>))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./dist&apos;</span>));
});

gulp.task(<span class="md-code-string">&apos;bump&apos;</span>, [<span class="md-code-string">&apos;build-shim&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">return</span> gulp.src([<span class="md-code-string">&apos;./package.json&apos;</span>, <span class="md-code-string">&apos;./bower.json&apos;</span>])
    .pipe(bump())
    .pipe(gulp.dest(<span class="md-code-string">&apos;./&apos;</span>));
});

gulp.task(<span class="md-code-string">&apos;tag&apos;</span>, [<span class="md-code-string">&apos;bump&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">()</span> </span>{
  <span class="md-code-keyword">var</span> pkg = <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./package.json&apos;</span>);
  <span class="md-code-keyword">var</span> v = <span class="md-code-string">&apos;v&apos;</span> + pkg.version;
  <span class="md-code-keyword">var</span> message = <span class="md-code-string">&apos;Release &apos;</span> + v;

  <span class="md-code-keyword">return</span> gulp.src(<span class="md-code-string">&apos;./&apos;</span>)
    .pipe(git.commit(message))
    .pipe(git.tag(v, message))
    .pipe(git.push(<span class="md-code-string">&apos;origin&apos;</span>, <span class="md-code-string">&apos;master&apos;</span>, <span class="md-code-string">&apos;--tags&apos;</span>))
    .pipe(gulp.dest(<span class="md-code-string">&apos;./&apos;</span>));
});

gulp.task(<span class="md-code-string">&apos;npm&apos;</span>, [<span class="md-code-string">&apos;tag&apos;</span>], <span class="md-code-function"><span class="md-code-keyword">function</span> <span class="md-code-params">(done)</span> </span>{
  <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;child_process&apos;</span>).spawn(<span class="md-code-string">&apos;npm&apos;</span>, [<span class="md-code-string">&apos;publish&apos;</span>], { stdio: <span class="md-code-string">&apos;inherit&apos;</span> })
    .on(<span class="md-code-string">&apos;close&apos;</span>, done);
});

gulp.task(<span class="md-code-string">&apos;test&apos;</span>, [<span class="md-code-string">&apos;lint&apos;</span>, <span class="md-code-string">&apos;mocha&apos;</span>]);
gulp.task(<span class="md-code-string">&apos;ci&apos;</span>, [<span class="md-code-string">&apos;build&apos;</span>]);
gulp.task(<span class="md-code-string">&apos;release&apos;</span>, [<span class="md-code-string">&apos;npm&apos;</span>]);
</code></pre> <p>You can also <a href="https://github.com/bevacqua/contra/blob/master/gulpfile.js" target="_blank" aria-label="gulpfile.js for bevacqua/contra on GitHub">check out the latest version</a> here.</p> <h1 id="bonus-track-integrating-gulp-with-travis-ci">Bonus Track: Integrating Gulp with Travis-CI</h1> <p>You just neeed to install <code class="md-code md-code-inline">gulp</code> in the <code class="md-code md-code-inline">before_install</code> section of your <code class="md-code md-code-inline">.travis.yml</code> manifest.</p> <pre class="md-code-block"><code class="md-code md-lang-yml">language: node_js

node_js:
  - 0.10
  - 0.11

before_install:
  - npm install -g gulp

script:
  - gulp ci
</code></pre> <p>Then, have a litte <code class="md-code md-code-inline">ci</code> task alias dedicated to your Continuous Integration platform, for example.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">gulp.task(<span class="md-code-string">&apos;ci&apos;</span>, [<span class="md-code-string">&apos;lint&apos;</span>, <span class="md-code-string">&apos;mocha&apos;</span>, <span class="md-code-string">&apos;build&apos;</span>]);
</code></pre> <p>If you&#x2019;ve never set up CI on Travis before, <a href="https://github.com/buildfirst/ci-by-example" target="_blank" aria-label="buildfirst/ci-by-example on GitHub">this short guide should help you</a>, even though it explains how to set it up with Grunt, the difference is really just in the <code class="md-code md-code-inline">.travis.yml</code> manifest contents.</p> <p>You can check out <a href="https://github.com/bevacqua/contra" target="_blank" aria-label="bevacqua/contra on GitHub">contra</a>, which is the package I&#x2019;ve been talking about in this article, for a working Gulpfile and integrated Travis-CI workflow.</p></div>
