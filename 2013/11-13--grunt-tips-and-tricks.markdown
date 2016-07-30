<h1>Grunt Tips and Tricks</h1>

<div><kbd>grunt</kbd> <kbd>tips</kbd></div>

<blockquote><p>I&#x2019;ve been meaning to compile a list of tips and tricks to improve you Grunt workflows, so here it is!</p><h6>In a Pinch</h6> <ul> <li>Always <code>--save-dev</code></li> <li>Heroku Custom Buildpack</li> <li>Forget <code>&#x2026;</code></li></ul></blockquote>

<div><p>I&#x2019;ve been meaning to compile a list of tips and tricks to improve you Grunt workflows, so here it is!</p></div>

<div></div>

<div><h6 id="in-a-pinch">In a Pinch</h6> <ul> <li>Always <code class="md-code md-code-inline">--save-dev</code></li> <li>Heroku Custom Buildpack</li> <li>Forget <code class="md-code md-code-inline">grunt.loadNpmTasks</code></li> <li>Spread out <code class="md-code md-code-inline">watch</code></li> <li>Use a nice JSHint reporter</li> <li>Keep your Gruntfile organized!</li> <li>Investigate</li> </ul> <p>These are explained and detailed below.</p></div>

<div><p><img alt="grunt.png" class="" src="https://i.imgur.com/EyXjS8r.png"></p> <h1 id="always-save-dev">Always <code class="md-code md-code-inline">--save-dev</code></h1> <p>Grunt should never be executed after deployments. Keeping build packages separated from the application&#x2019;s dependencies is best practice.</p> <h1 id="heroku-custom-buildpack">Heroku Custom Buildpack</h1> <p>Heroku isn&#x2019;t an excuse to place Grunt packages as <code class="md-code md-code-inline">dependencies</code>, either. Simply use <a href="https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt" target="_blank" aria-label="heroku-buildpack-nodejs-grunt on GitHub">a custom buildpack</a> to solve this.</p> <p>For existing applications, you need to add an environment variable:</p> <pre class="md-code-block"><code class="md-code md-lang-bash">heroku config:add BUILDPACK_URL=https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git
</code></pre> <p>For new applications, just specify the buildpack:</p> <pre class="md-code-block"><code class="md-code md-lang-bash">heroku create myapp --buildpack https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git
</code></pre> <p>Next, create a <code class="md-code md-code-inline">&apos;heroku&apos;</code> task alias, which will run the build on the Heroku servers. If the build fails, the application won&#x2019;t get deployed, this doubles as some sort of CI safeguard.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">grunt.registerTask(<span class="md-code-string">&apos;heroku&apos;</span>, [<span class="md-code-string">&apos;jshint&apos;</span>, <span class="md-code-string">&apos;build&apos;</span>]);
</code></pre> <p>Check out <a href="https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt" target="_blank" aria-label="heroku-buildpack-nodejs-grunt on GitHub">the buildpack&#x2019;s documentation</a> if you need it to work <em>in different environments</em>.</p> <h1 id="forget-gruntloadnpmtasks">Forget <code class="md-code md-code-inline">grunt.loadNpmTasks</code></h1> <p><img alt="tasks.png" class="" src="https://i.imgur.com/9SCtIYz.png"></p> <p>Use <a href="https://github.com/sindresorhus/load-grunt-tasks" target="_blank" aria-label="load-grunt-tasks on GitHub">load-grunt-tasks</a> to load your Grunt plugins.</p> <pre class="md-code-block"><code class="md-code md-lang-bash">npm i --save-dev load-grunt-tasks
</code></pre> <p>Then, rather than all the calls to <code class="md-code md-code-inline">grunt.loadNpmTask()</code>, use this one liner.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;load-grunt-tasks&apos;</span>)(grunt);
</code></pre> <p>Never forget to register a Grunt plugin again!</p> <h1 id="spread-out-your-watch">Spread out your <code class="md-code md-code-inline">watch</code></h1> <p>Instead of single monolithic <code class="md-code md-code-inline">watch</code> target that rebuilds the entire project, use targets to run specific parts of your build, reducing the reactions to little bursts, instead of painfully slow builds.</p> <p>Detailed example:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">watch: {
    rebuild: {
    	tasks: [<span class="md-code-string">&apos;build:rebuild&apos;</span>],
    	files: [<span class="md-code-string">&apos;Gruntfile.js&apos;</span>, <span class="md-code-string">&apos;build/**/*.js&apos;</span>]
    },
    jshint_client: {
    	tasks: [<span class="md-code-string">&apos;jshint:client&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/js/**/*.js&apos;</span>]
    },
    jshint_client_tests: {
    	tasks: [<span class="md-code-string">&apos;jshint:client_tests&apos;</span>],
    	files: [<span class="md-code-string">&apos;test/client/**/*.js&apos;</span>]
    },
    jshint_server: {
    	tasks: [<span class="md-code-string">&apos;jshint:server&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/srv/**/*.js&apos;</span>, <span class="md-code-string">&apos;app.js&apos;</span>]
    },
    jshint_server_tests: {
    	tasks: [<span class="md-code-string">&apos;jshint:server_tests&apos;</span>],
    	files: [<span class="md-code-string">&apos;test/server/**/*.js&apos;</span>]
    },
    jshint_server_support: {
    	tasks: [<span class="md-code-string">&apos;jshint:server_support&apos;</span>],
    	files: [<span class="md-code-string">&apos;Gruntfile.js&apos;</span>, <span class="md-code-string">&apos;build/**/*.js&apos;</span>, <span class="md-code-string">&apos;deploy/**/*.js&apos;</span>]
    },
    test_client: {
    	tasks: [<span class="md-code-string">&apos;karma:unit_background:run&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/js/**/*.js&apos;</span>, <span class="md-code-string">&apos;test/client/**/*.js&apos;</span>]
    },
    test_server: {
    	tasks: [<span class="md-code-string">&apos;mochaTest:unit&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/srv/**/*.js&apos;</span>, <span class="md-code-string">&apos;app.js&apos;</span>, <span class="md-code-string">&apos;test/server/**/*.js&apos;</span>]
    },
    images: {
    	tasks: [<span class="md-code-string">&apos;images:debug&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/img/**/*.{png,jpg,gif,ico}&apos;</span>]
    },
    css: {
    	tasks: [<span class="md-code-string">&apos;css:debug&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/css/**/*.styl&apos;</span>, <span class="md-code-string">&apos;bin/.tmp/sprite/*.css&apos;</span>, <span class="md-code-string">&apos;bower_components/**/*.css&apos;</span>]
    },
    js_sources: {
    	tasks: [<span class="md-code-string">&apos;copy:js_sources&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/js/**/*.js&apos;</span>]
    },
    js_bower: {
    	tasks: [<span class="md-code-string">&apos;copy:js_bower_debug&apos;</span>],
    	files: [<span class="md-code-string">&apos;bower_components/**/*.js&apos;</span>]
    },
    views: {
    	tasks: [<span class="md-code-string">&apos;views:debug&apos;</span>],
    	files: [<span class="md-code-string">&apos;src/client/views/**/*.jade&apos;</span>]
    },
    livereload: {
    	options: { livereload: <span class="md-code-literal">true</span> },
    	files: [<span class="md-code-string">&apos;bin/public/**/*.{css,js}&apos;</span>,<span class="md-code-string">&apos;bin/views/**/*.html&apos;</span>]
    }
}
</code></pre> <h1 id="use-a-nice-jshint-reporter">Use a nice JSHint reporter</h1> <p>The one I&#x2019;m currently using is <a href="https://github.com/sindresorhus/jshint-stylish" target="_blank" aria-label="jshint-stylish on GitHub">jshint-stylish</a>, which is pretty nice and easy to configure. For example:</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">grunt.initConfig({
  jshint: {
    options: {
      reporter: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;jshint-stylish&apos;</span>)
    },
    foo: {
      files: [<span class="md-code-string">&apos;bar.js&apos;</span>]
    }
  }
});
</code></pre> <p>Done, pretty JSHint reports!</p> <p><img alt="reporter.png" class="" src="https://github.com/sindresorhus/jshint-stylish/raw/master/screenshot.png"></p> <h1 id="keep-your-gruntfile-organized">Keep your Gruntfile organized!</h1> <p>I like to separate my configuration in different files for each workflow: development, release, and deployment. You could use conventions and stuff like that to load different files, but I generally lean towards manually using <code class="md-code md-code-inline">require</code>, and then merging the configuration together with <a href="http://lodash.com/" target="_blank" aria-label="Next Generation Underscore">Lo-Dash</a>&#x2019;s <code class="md-code md-code-inline">_.merge</code>.</p> <p>Use the following line in your Gruntfile.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript">grunt.initConfig(_.merge.apply({}, _.values(<span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./build/cfg&apos;</span>))));
</code></pre> <p>Then, do something like this in your <code class="md-code md-code-inline">./build/cfg/index.js</code> file. Keep in mind the keys (<code class="md-code md-code-inline">manifest</code>, <code class="md-code md-code-inline">dev</code>, <code class="md-code md-code-inline">env</code>, etc) aren&#x2019;t important.</p> <pre class="md-code-block"><code class="md-code md-lang-javascript"><span class="md-code-built_in">module</span>.exports = {
    manifest: {
        pkg: grunt.file.readJSON(<span class="md-code-string">&apos;package.json&apos;</span>)
    },
    dev: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./task/development.js&apos;</span>),
    env: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./task/environment.js&apos;</span>),
    build: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./task/build.js&apos;</span>),
    release: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./task/release.js&apos;</span>),
    deploy: <span class="md-code-built_in">require</span>(<span class="md-code-string">&apos;./task/deploy.js&apos;</span>)
};
</code></pre> <p>In each of those modules, configure any tasks you need to configure for that workflow. Keep in mind that <strong>merge works recursively</strong>. This enables you to configure different targets for the same task in different files!</p> <h1 id="investigate">Investigate</h1> <p>If something feels too clunky, ask around. Chances are, there is a better way to do things. My <a href="https://github.com/bevacqua/unbox" target="_blank" aria-label="unbox on GitHub">unbox</a> project follows many of these practices, and a few more. You might want to check it out while in investigation mode.</p></div>
