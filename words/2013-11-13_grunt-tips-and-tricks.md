# Grunt Tips and Tricks

I've been meaning to compile a list of tips and tricks to improve you Grunt workflows, so here it is!

###### In a Pinch

- Always `--save-dev`
- Heroku Custom Buildpack
- Forget `grunt.loadNpmTasks`
- Spread out `watch`
- Use a nice JSHint reporter
- Keep your Gruntfile organized!
- Investigate

These are explained and detailed below.

![grunt.png][1]

# Always `--save-dev`

Grunt should never be executed after deployments. Keeping build packages separated from the application's dependencies is best practice.

# Heroku Custom Buildpack

Heroku isn't an excuse to place Grunt packages as `dependencies`, either. Simply use [a custom buildpack](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt "heroku-buildpack-nodejs-grunt on GitHub") to solve this.

For existing applications, you need to add an environment variable:

```shell
heroku config:add BUILDPACK_URL=https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git
```

For new applications, just specify the buildpack:

```shell
heroku create myapp --buildpack https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt.git
```

Next, create a `'heroku'` task alias, which will run the build on the Heroku servers. If the build fails, the application won't get deployed, this doubles as some sort of CI safeguard.

```js
grunt.registerTask('heroku', ['jshint', 'build']);
```

Check out [the buildpack's documentation](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt "heroku-buildpack-nodejs-grunt on GitHub") if you need it to work _in different environments_.

# Forget `grunt.loadNpmTasks`

![tasks.png][2]

Use [load-grunt-tasks](https://github.com/sindresorhus/load-grunt-tasks "load-grunt-tasks on GitHub") to load your Grunt plugins.

```shell
npm i --save-dev load-grunt-tasks
```

Then, rather than all the calls to `grunt.loadNpmTask()`, use this one liner.

```js
require('load-grunt-tasks')(grunt);
```

Never forget to register a Grunt plugin again!

# Spread out your `watch`

Instead of single monolithic `watch` target that rebuilds the entire project, use targets to run specific parts of your build, reducing the reactions to little bursts, instead of painfully slow builds.

Detailed example:

```js
watch: {
    rebuild: {
    	tasks: ['build:rebuild'],
    	files: ['Gruntfile.js', 'build/**/*.js']
    },
    jshint_client: {
    	tasks: ['jshint:client'],
    	files: ['src/client/js/**/*.js']
    },
    jshint_client_tests: {
    	tasks: ['jshint:client_tests'],
    	files: ['test/client/**/*.js']
    },
    jshint_server: {
    	tasks: ['jshint:server'],
    	files: ['src/srv/**/*.js', 'app.js']
    },
    jshint_server_tests: {
    	tasks: ['jshint:server_tests'],
    	files: ['test/server/**/*.js']
    },
    jshint_server_support: {
    	tasks: ['jshint:server_support'],
    	files: ['Gruntfile.js', 'build/**/*.js', 'deploy/**/*.js']
    },
    test_client: {
    	tasks: ['karma:unit_background:run'],
    	files: ['src/client/js/**/*.js', 'test/client/**/*.js']
    },
    test_server: {
    	tasks: ['mochaTest:unit'],
    	files: ['src/srv/**/*.js', 'app.js', 'test/server/**/*.js']
    },
    images: {
    	tasks: ['images:debug'],
    	files: ['src/client/img/**/*.{png,jpg,gif,ico}']
    },
    css: {
    	tasks: ['css:debug'],
    	files: ['src/client/css/**/*.styl', 'bin/.tmp/sprite/*.css', 'bower_components/**/*.css']
    },
    js_sources: {
    	tasks: ['copy:js_sources'],
    	files: ['src/client/js/**/*.js']
    },
    js_bower: {
    	tasks: ['copy:js_bower_debug'],
    	files: ['bower_components/**/*.js']
    },
    views: {
    	tasks: ['views:debug'],
    	files: ['src/client/views/**/*.jade']
    },
    livereload: {
    	options: { livereload: true },
    	files: ['bin/public/**/*.{css,js}','bin/views/**/*.html']
    }
}
```

# Use a nice JSHint reporter

The one I'm currently using is [jshint-stylish](https://github.com/sindresorhus/jshint-stylish "jshint-stylish on GitHub"), which is pretty nice and easy to configure. For example:

```js
grunt.initConfig({
  jshint: {
    options: {
      reporter: require('jshint-stylish')
    },
    foo: {
      files: ['bar.js']
    }
  }
});
```

Done, pretty JSHint reports!

![reporter.png][3]

# Keep your Gruntfile organized!

I like to separate my configuration in different files for each workflow: development, release, and deployment. You could use conventions and stuff like that to load different files, but I generally lean towards manually using `require`, and then merging the configuration together with [Lo-Dash](http://lodash.com/ "Next Generation Underscore")'s `_.merge`.

Use the following line in your Gruntfile.

```js
grunt.initConfig(_.merge.apply({}, _.values(require('./build/cfg'))));
```

Then, do something like this in your `./build/cfg/index.js` file. Keep in mind the keys (`manifest`, `dev`, `env`, etc) aren't important.

```js
module.exports = {
    manifest: {
        pkg: grunt.file.readJSON('package.json')
    },
    dev: require('./task/development.js'),
    env: require('./task/environment.js'),
    build: require('./task/build.js'),
    release: require('./task/release.js'),
    deploy: require('./task/deploy.js')
};
```

In each of those modules, configure any tasks you need to configure for that workflow. Keep in mind that **merge works recursively**. This enables you to configure different targets for the same task in different files!

# Investigate

If something feels too clunky, ask around. Chances are, there is a better way to do things. My [unbox](https://github.com/bevacqua/unbox "unbox on GitHub")  project follows many of these practices, and a few more. You might want to check it out while in investigation mode.

  [1]: http://i.imgur.com/EyXjS8r.png
  [2]: http://i.imgur.com/9SCtIYz.png
  [3]: https://github.com/sindresorhus/jshint-stylish/raw/master/screenshot.png