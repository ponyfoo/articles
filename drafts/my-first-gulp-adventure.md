# My First Gulp Adventure

I decided to take a gulp of Gulp and use it in one of my latest projects, to help me with releases. I wrote a Gulpfile, which lets me write some code to define the tasks enumerated below.

- Lint my source code
- Run unit tests
- Clean my distribution directory
- Build the distribution files, minified and otherwise
- Get the file size of both the regular and minified versions
- Bump the package version for `npm` and `bower`
- Push a new tag to `git`, to update the Bower version
- Publish the updated version to `npm`

In this article I aim to explain what I did, how I did it, and the reasons why I made some of the choices that I did. The only real problem I had had to do with synchronicity. I felt it would be interesting walking you through the process. It may help you get started with Gulp!

![rocket.png][1]

  [1]: http://i.imgur.com/ApIcjlI.png "The Gulp Rocket!"

# Gulp's Dependency System

It seems that [they haven't yet settled][1] on an approach to tasks depending on other tasks Some people write their own versions of how this should look like, while the author suggests we use the "dependency system". Running things in parallel sounds appealing on paper, but it loses value as you realize you probably want to log output serially, in order to be able to make sense out of it. A tempting possibility might be buffering the output produced by each task, and then flushing it as each task finishes, but this feels like too convoluted to ever be elegant, or a thing.

By default, Gulp tasks run asynchronously, and the "dependency system", allows you to fire pre-requisite tasks before a given task is able to run. **It feels verbose and unlike the rest of Gulp**.

```js
gulp.task('one', function (cb) {
  cb();
});

gulp.task('two', ['one'], function (cb) {
  cb();
});

gulp.task('three', ['two'], function (cb) {
  cb();
});

gulp.task('default', ['three']);
```

If you know of _a better way_, within Gulp, please let me know! I imagine running tasks in series is a pretty darn common thing, and I don't understand the reason why they made the task runner work asynchronously in the first place.

That being said, the rest of the `'use gulp';` experience was pretty awesome. Let's delve into that!

# Getting Started

First things first, you'll need to install Gulp both globally, just one time; and locally, for your package.

```shell
npm i -g gulp
npm i -D gulp
```

Next, you create a `gulpfile.js` file which looks like below.

```js
var gulp = require('gulp');

gulp.task('default', function () {
});
```

Great. You have your default task, and it does a whole lot of nothing. _Asynchronously!_

# Building your package

Let's have that build and minify our code. We'll need to install a few `npm` packages to do that.

```js
npm i -D gulp-uglify gulp-concat gulp-rename
```

Here's how you would create your first task, which bundles, and minifies our source code, writing the results to disk.

```js
var gulp = require('gulp');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');
var pkg = require('./package.json');

gulp.task('build', function () {
  return gulp.src('./src/*.js')
    .pipe(concat(pkg.name + '.js'))
    .pipe(gulp.dest('./dist'))
    .pipe(rename(pkg.name + '.min.js'))
    .pipe(uglify())
    .pipe(gulp.dest('./dist'));
});
```

Let's go line by line to get a better feel of what we're doing.

### Some `require` statements

Here we're just getting the plugin packages, nothing special. Just regular [Common.JS] `require` statements. Let's skip those lines, since they don't add much.

```js
var gulp = require('gulp');
var concat = require('gulp-concat');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');
```

Contrary to how Grunt operated, we need to `require('gulp')` in our Gulpfiles, which I think is better than exporting a function, like we do when using Grunt, as it's _just unnecessary_.

### `var pkg = require('./package.json');`

I'm just leveraging `package.json` metadata in order to be able to use the package name as the file name, which would allow me to copy and paste this Gulpfile into some other project, with similar requirements, and not even need to edit the build!

### `gulp.task('build', function () {`

Here I'm defining a `build` task, which can be required by other tasks as a pre-requisite, executed as a sub-task using `gulp.run('build')`, or executed via the CLI using `gulp build`. Since I'm not taking a `cb` argument, I should return the build stream, so that I can make the task a dependency which would finish when the stream is closed.

### `return gulp.src('./src/*.js')`

The `return` statement signals that we want the task to run synchronously. In other words, the task won't _"finish"_ immediately, and thus it will effectively block other tasks if we were to add it as a dependency, until the stream is closed.

> Note that `gulp.run` has been [deprecated in Gulp `v3.5`][7] and will be [removed entirely in Gulp `v4`][8].

The examples below would wait until our `'build'` task is completed.

```js
gulp.run('build', function () {
  // build is complete!
});
```

```js
gulp.task('release', ['build'], function () {
  // build is complete, release the kraken!
});
```

Lastly, `gulp.src('./src/*.js')` tells Gulp that it'll have to work with the files which [match the globbing expression][3] `'./src/*.js'`, or all JavaScript files in the `./src` directory. At this point, however, Gulp doesn't know anything else, only that it needs to work with those source files.

### `.pipe(concat(pkg.name + '.js'))`

Here things start getting interesting. First, `concat(pkg.name + '.js')` [constructs a stream][4] that will bundle together all files piped into it. Then, `.pipe()` will do exactly that, pipe the source files chosen matching the globbing expression in the previous step. This results in source files getting bundled [into a single data blob][5].

### `.pipe(gulp.dest('./dist'))`

Up until this point, everything has been done in memory. The `gulp.dest('./dist')` statement returns a write stream which writes to disk, in the `./dist` directory. Once the `concat` operation is completed, a single data blob will be piped into the `dest` stream, writing the bundle to disk.

### `.pipe(rename(pkg.name + '.min.js'))`

Doing `rename(pkg.name + '.min.js')` creates a stream which [changes the destination filename][6] from what we originally set when creating the `concat()` stream. Subsequent calls to `.dest()` will be told to write to this filename, instead.

### `.pipe(uglify())`

You can probably guess that `uglify()` creates a stream, which minifies the bundle and emits that.

### `.pipe(gulp.dest('./dist'));`

Lastly, we pipe into `./dist` again, writing bundled, minified code, into a single file.

So that's it, let's look at that task again.

```js
gulp.task('build', function () {
  return gulp.src('./src/*.js')
    .pipe(concat(pkg.name + '.js'))
    .pipe(gulp.dest('./dist'))
    .pipe(rename(pkg.name + '.min.js'))
    .pipe(uglify())
    .pipe(gulp.dest('./dist'));
});
```

I'd like to see the minified file size every time I run this task, and I could use `gulp-size` to do that. Let's see

### `.pipe(size())`

First, we need to install `gulp-size`.

```shell
npm i -D gulp-size
```

Then, we just add that stream to our pipeline, right after we uglify (minify) our code.

```js
gulp.task('build', function () {
  return gulp.src('./src/*.js')
    .pipe(concat(pkg.name + '.js'))
    .pipe(gulp.dest('./dist'))
    .pipe(rename(pkg.name + '.min.js'))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest('./dist'));
});
```

Don't forget to `require` it!

# Pushing a release

Once you're able to build your package, you'd probably want to automate the tedious process of releasing a package update. For example, here's everything that happens when I release a new version of [`contra`][9].

- I build the regular and minified library versions
- I run unit tests to ensure everything is working as expected
- I bump the version number in both `package.json` and `bower.json`
- I create a commit with those changes
- I create a tag for the release
- I push those changes so that Bower can tell I updated the library
- I publish an update to `npm`

Yeah, that ain't gonna work if I'd like to push several updates in short succession for any reason, or if I have to maintain any more libraries. And even if I don't, doing all of that by hand introduces the very likely possibility that I make a mistake, or forget one of the steps, resulting in unhappy package consumers.

> No, it's **better to automate releases.**

Imagine if I were able to just do `gulp release` and have all of that happen. Actually, that's _exactly how it is set up._ In one task, I bump the package version, I do all the git-related operations in another, and I push to `npm` in a third task. Let's dissect each of them.

## Bumping packages

Bumping the package version is pretty straightforward, and we can use [`gulp-bump`][10], which does that, and only that.

```js
gulp.task('bump', function () {
  return gulp.src(['./package.json', './bower.json'])
    .pipe(bump())
    .pipe(gulp.dest('./'));
});
```

I don't even have to decompose that one, you just tell Gulp to read from `package.json` and `bower.json`, or whichever versioned JSON manifests you have, and pipe that through `bump()` and into the `dest` write stream. Easy peasy!

Just remember to install and `require` [gulp-bump][11].

```shell
npm i -D gulp-bump
```

```js
var bump = require('gulp-bump');
```

## Tagging on `git`

There's an awesome `git` package for Gulp in [`gulp-git`][11]. It does everything. It commits, it tags, it pushes, and everything else. Seriously, [go look at it's documentation][11]. Terrific! The author [documented it like a gentleman][12], good stuff.

In this particular task, I chose to use the `package.json` data again. I use it to sign my commit with the release number. Then I push the `master` branch to the `origin` remote, and I include the `--tags`, so that I don't have to do that by hand either.

```js
gulp.task('tag', function () {
  var pkg = require('./package.json');
  var v = 'v' + pkg.version;
  var message = 'Release ' + v;

  return gulp.src('./')
    .pipe(git.commit(message))
    .pipe(git.tag(v, message))
    .pipe(git.push('origin', 'master', '--tags'))
    .pipe(gulp.dest('./'));
});
```

Being able to release just like that is pretty awesome. However, this task depends directly on the bump task in order to succeed, as it'll use the version number to create the tag. We'll get into the flow later, for now this just works if we run them like below.

```shell
gulp bump
gulp tag
```

Soon we'll check out how these dependencies can be sorted out.

## Publishing on `npm`

I couldn't find a `gulp-npm` package for my `npm publish` purposes, so I just created my own task, without developing a full-fledged plugin.

```js
gulp.task('npm', function (done) {
  require('child_process').spawn('npm', ['publish'], { stdio: 'inherit' })
    .on('close', done);
});
```

In case you've never seen it before, setting `{ stdio: 'inherit' }` when spawning a child process, then the child will use your standard input, output, and error. In other words, `npm publish` will be able to print its output on your terminal when you run `gulp npm`.

### Putting it all together

To put it all together, all that's required is adding an array with the dependencies to each task. Here's is [contra][9]'s complete Gulpfile.

```js
var gulp = require('gulp');
var bump = require('gulp-bump');
var git = require('gulp-git');
var jshint = require('gulp-jshint');
var mocha = require('gulp-mocha');
var clean = require('gulp-clean');
var rename = require('gulp-rename');
var uglify = require('gulp-uglify');
var size = require('gulp-size');

gulp.task('lint', function () {
  return gulp.src('./src/*.js')
    .pipe(jshint('.jshintrc'))
    .pipe(jshint.reporter('jshint-stylish'));
});

gulp.task('mocha', function () {
  gulp.src('./test/*.js')
    .pipe(mocha({ reporter: 'list' }));
});

gulp.task('clean', function () {
  return gulp.src('./dist', { read: false })
    .pipe(clean());
});

gulp.task('build', ['test', 'clean'], function () {
  return gulp.src('./src/contra.js')
    .pipe(gulp.dest('./dist'))
    .pipe(rename('contra.min.js'))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest('./dist'));
});

gulp.task('build-shim', ['build'], function () {
  return gulp.src('./src/contra.shim.js')
    .pipe(gulp.dest('./dist'))
    .pipe(rename('contra.shim.min.js'))
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest('./dist'));
});

gulp.task('bump', ['build-shim'], function () {
  return gulp.src(['./package.json', './bower.json'])
    .pipe(bump())
    .pipe(gulp.dest('./'));
});

gulp.task('tag', ['bump'], function () {
  var pkg = require('./package.json');
  var v = 'v' + pkg.version;
  var message = 'Release ' + v;

  return gulp.src('./')
    .pipe(git.commit(message))
    .pipe(git.tag(v, message))
    .pipe(git.push('origin', 'master', '--tags'))
    .pipe(gulp.dest('./'));
});

gulp.task('npm', ['tag'], function (done) {
  require('child_process').spawn('npm', ['publish'], { stdio: 'inherit' })
    .on('close', done);
});

gulp.task('test', ['lint', 'mocha']);
gulp.task('ci', ['build']);
gulp.task('release', ['npm']);
```

You can also [check out the latest version][15] here.

# Bonus Track: Integrating Gulp with Travis-CI

You just neeed to install `gulp` in the `before_install` section of your `.travis.yml` manifest.

```yml
language: node_js

node_js:
  - 0.10
  - 0.11

before_install:
  - npm install -g gulp

script:
  - gulp ci
```

Then, have a litte `ci` task alias dedicated to your Continuous Integration platform, for example.

```js
gulp.task('ci', ['lint', 'mocha', 'build']);
```

If you've never set up CI on Travis before, [this short guide should help you][14], even though it explains how to set it up with Grunt, the difference is really just in the `.travis.yml` manifest contents.

You can check out [contra][9], which is the package I've been talking about in this article, for a working Gulpfile and integrated Travis-CI workflow.

  [1]: https://github.com/gulpjs/gulp/issues/96 "'Support running task synchronously' - issue on GitHub"
  [2]: http://wiki.commonjs.org/wiki/Modules/1.1 "Common.JS modules"
  [3]: http://gruntjs.com/configuring-tasks#globbing-patterns "Globbing Patterns Explained"
  [4]: https://github.com/wearefractal/gulp-concat/blob/master/index.js#L43 "The stream returned by concat"
  [5]: https://github.com/wearefractal/gulp-concat/blob/master/index.js#L39 "Data emitted by the concat stream"
  [6]: https://github.com/hparra/gulp-rename/blob/master/index.js#L41 "Renaming the destination filename"
  [7]: https://github.com/gulpjs/gulp/blob/aea0f93eaae9204a0a42e8e83372266915b089b5/CHANGELOG.md#35 "Gulp Changes in v3.5"
  [8]: https://github.com/gulpjs/gulp/blob/aea0f93eaae9204a0a42e8e83372266915b089b5/index.js#L16 "gulp.run removal in v4"
  [9]: https://github.com/bevacqua/contra "bevacqua/contra on GitHub"
  [10]: https://github.com/stevelacy/gulp-bump "stevelacy/gulp-bump on GitHub"
  [11]: https://github.com/stevelacy/gulp-git "stevelacy/gulp-git on GitHub"
  [12]: /2014/01/20/how-to-design-great-programs "How to Design Great Programs"
  [13]: http://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options "child_process.spawn(command, [args], [options])"
  [14]: https://github.com/buildfirst/ci-by-example "buildfirst/ci-by-example on GitHub"
  [15]: https://github.com/bevacqua/contra/blob/master/gulpfile.js "gulpfile.js for bevacqua/contra on GitHub"
