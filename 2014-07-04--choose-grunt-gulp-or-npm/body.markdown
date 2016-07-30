As a first step, let's discuss where Grunt excels at.

# Grunt: The Good Parts

The single best aspect of [Grunt][2] is its ease of use. It enables programmers to develop build flows using JavaScript almost effortlessly. All that's required is searching for the appropriate plugin, reading its documentation, and then installing and configuring it. This ease of use means that members of large development teams, who are often of varying skill levels, don't have any trouble tweaking the build flow to meet the latest needs of the project. The team doesn't need to be fluent in Node either, they need to add properties to the configuration object, and task names to the different arrays that make up the build flow.

There's a plugin base large enough that you'll rarely find yourself needing to develop your own build tasks, which also enables you and your team to rapidly develop a build process, which is crucial if you're going for a [Build First][0] approach, even when taking small steps and progressively developing your build flows.

It's also feasible to manage deployments through Grunt, as many packages exist to accommodate for those tasks, such as `grunt-git`, `grunt-rsync`, or [`grunt-ec2`][4], to name a few.

So where does Grunt fall short? It may get too verbose if you have a significantly large build flow. It's often hard to make sense of the build flow as a whole once it has been in development for a while. Once the task count in your build flows gets to the double digits, it's almost guaranteed that you'll find yourself having to run targets which belong to the same task individually, so that you're able to compose the flow in the right order. Since tasks are configured declaratively, you'll also have a hard time figuring out the order in which tasks get executed.

Besides that, your team should be dedicated to writing maintainable code when it comes to your builds as well, and in the case of Grunt that means maintaining separate files for the configuration of each task, or at least for each of the build flows that your team uses.

Now that we've identified the good and the bad in Grunt, as well as the situations in which it might be a better fit for your project, let's talk about npm, how it can be leveraged as a build tool, and its differences with Grunt.

# npm as a build tool

In order to use npm as a build tool, you'll need a `package.json` file and `npm` itself. Defining tasks for npm is as easy as adding properties to a `scripts` object in your package manifest. The name of the property will be used as the task name, and the value will be the command you want to execute. The example shown below uses the JSHint command-line interface to run a linter through our JavaScript files and check for errors. You can run any shell command that you need.

```json
{
  "scripts": {
    "test": "jshint . --exclude node_modules"
  },
  "devDependencies": {
    "jshint": "^2.5.1"
  }
}
```

Once the task is defined, it can be executed in your command-line by running the following command.

```
npm run test
```

Note that npm provides shortcuts for specific task names. In the case of `test`, you could simply do `npm test` and omit the `run` verb. You can compose build flows by chaining `npm run` commands together in your script declarations.

```json
{
  "scripts": {
    "lint": "jshint . --exclude node_modules",
    "unit": "tape test/*",
    "test": "npm run lint && npm run unit"
  },
  "devDependencies": {
    "jshint": "^2.5.1",
    "tape": "~2.10.2"
  }
}
```

You could also schedule tasks as background jobs, making them asynchronous. Suppose we have the following package file, where we'll just copy a directory in our JavaScript build flow, and compile an Stylus stylesheet during our CSS build flow (Stylus is a CSS preprocessor). In this case, running the tasks asynchronously is ideal. You can achieve that using `&` as a separator, or after a command.

```json
{
  "scripts": {
    "build-js": "cp -r src/js/vendor bin/js",
    "build-css": "stylus src/css/all.styl -o bin/css",
    "build": "npm run build-js & npm run build-css"
  },
  "devDependencies": {
    "stylus": "^0.45.0"
  }
}
```

To learn more about npm as a build tool, you should try and learn how to write Bash commands instead.

## Installing npm task dependencies

The JSHint CLI is not necessarily available in your system, and there's two ways you could go about installing it. If you're looking to use the tool directly from your command-line, and not in an `npm run` task, then you should install it globally, using the `-g` flag as shown below.

```shell
npm install -g jshint
```

However, if you're using the package in an `npm run` task, then you should be adding it as a `devDependency`, as shown below. That'll allow npm to find the JSHint package on any system where the package dependencies are installed, rather than expecting the environment to have JSHint installed globally. This applies to any CLI tools that aren't readily available in operating systems.

```shell
npm install --save-dev jshint
```

You aren't limited to using just CLI tools. In fact, npm is able to run any shell script. Let's dig into that!

## Using shell scripts in npm tasks

Below is an example script that runs on Node, and displays a random emoji string. The first line tells the environment that the script is in Node.

```shell
#!/usr/bin/env node

var emoji = require('emoji-random');
var emo = emoji.random();

console.log(emo);
```

If you were to place that script in a file named `emoji` at the root of our project, you'd have to declare `emoji-random` as a dependecy and add the command to the `scripts` object in the package manifest.

```json
{
  "scripts": {
    "emoji": "./emoji"
  },
  "devDependencies": {
    "emoji-random": "^0.1.2"
  }
}
```

Once that's out of the way, running the command is merely a matter of invoking `npm run emoji` in your terminal.

## The good and the bad

Using [npm as a build tool][3] has several advantages over Grunt. You aren't constrained to Grunt plugins, and thus you can take advantage of all of npm, which hosts tens of thousands of packages. You won't need any additional CLI tooling or files other than `npm`, which you are already using to manage dependencies, and your `package.json` manifest, where dependencies and your build commands are listed. Since `npm` runs CLI tools and Bash commands directly, it'll perform way better than Grunt could.

Take into account that one of the biggest disadvantages of Grunt is the fact that it's I/O bound. This means that most Grunt tasks read from disk, and then write to disk. If you have several tasks working on the same files, then chances are the file is going to be read from disk multiple times. In Bash, commands can pipe the output of a command directly into the next one, avoiding the extra I/O overhead in Grunt.

Probably the biggest disadvantage to npm is the fact that Bash doesn't play all that well with Windows environments. This means that open-source projects using `npm run` might run into issues when people try to fiddle with them on Windows. In a similar light, it also means that Windows developers will try and use alternatives to `npm`, instead. That drawback pretty much rules out `npm` for projects that need to be able to run on Windows.

[Gulp][1], another build tool, presents similarities with both Grunt and npm, as you'll discover in a moment.

# Gulp, the streaming build tool

Gulp is similar to Grunt in that it relies on plugins and it's cross-platform, supporting Windows users as well. Gulp is a code-driven build tool, in contrast with Grunt's declarative approach to task definition, making your task definitions a bit easier to read. Gulp is also similar to `npm run` in that it uses Node streams to read files and pipe data through functions that transform it into output that will end up being written to disk. This means that Gulp doesn't have the disk-intensive I/O issues that you may observe in using Grunt. It's also faster than Grunt for the same reason, less time spent in I/O.

The main disadvantage in using Gulp is that it relies heavily on streams, pipes, and asynchronous code. Don't get me wrong: if you're into Node, then that's definitely an advantage as well. But the issue with those concepts is that unless you and your team are well-versed in Node, you're probably going to run into issues dealing with streams, especially if you have to build your own Gulp task plugins.

When working in teams, Gulp is not as prohibitive as `npm`, because most of your front-end team probably knows JavaScript, while chances are they're not that fluent in Bash scripting, and some of them may be using Windows! That's why I usually suggest to keep `npm run` to your personal projects, maybe use Gulp in projects where the team is comfy with Node, and Grunt everywhere else. Of course, that's my personal appreciation, you should think for yourself and figure out what works best for you and your team. Also, you shouldn't constrain yourself to Grunt, Gulp, or `npm run` just because those are the tools that work for me. Try and do a little research, and maybe you'll find a tool that you like even better than those three.

Let's walk through some examples to get a feel of how Gulp tasks look like.

## Running tests in Gulp

Gulp is quite similar to Grunt in its conventions. In Grunt there's a `Gruntfile.js` file, used to define your build tasks, and in Gulp the file needs to be named `Gulpfile.js` instead. The other minor difference is that in the case of Gulp, the CLI is contained in the same package as the task runner, so you'll have to install the `gulp` package from npm both locally and globally.

```shell
touch Gulpfile.js
npm install -g gulp
npm install --save-dev gulp
```

To get started, I'll create a Gulp task to lint a JavaScript file, using JSHint just like you've already seen with Grunt and `npm run`. In the case of Gulp, you'll have to install the `gulp-jshint` Gulp plugin for JSHint.

```shell
npm install --save-dev gulp-jshint
```

Now that you're fully equipped with the CLI, that you globally installed, the local `gulp` instalation, and the `gulp-jshint` plugin, you can put together the build task to run the linter. To define build tasks with Gulp, you'll have to write them programatically in the `Gulpfile.js` file.

First off you'll use `gulp.task` passing it a task name and a function. The function contains all of the code necessary to run that task. Here you should use `gulp.src` to create a read stream into your source files, using a globbing pattern like the ones you've seen in our experiences learning about Grunt. That same stream should be piped into the JSHint plugin, which you can configure or just use with the defaults it comes with. Then all you'd have to do is pipe the results of the JSHint task through a reporter, and have it print the results to your terminal. All of what I've just described results in the Gulpfile presented below.

```js
var gulp = require('gulp');
var jshint = require('gulp-jshint');

gulp.task('test', function () {
  return gulp
    .src('./sample.js')
    .pipe(jshint())
    .pipe(jshint.reporter('default'));
});
```

I should also mention that I'm returning the stream so that Gulp understands that it should wait for the data to stop flowing before it considers the task to be completed. You could use a custom JSHint reporter in order to have the output be a bit more concise, and thus easier to read by humans. JSHint reporters don't need to be Gulp plugins, so you could use `jshint-stylish` for example. Let's install it locally.

```js
npm install --save-dev jshint-stylish
```

The updated Gulpfile should look as shown below. It'll load the `jshint-stylish` module to format the reporting output.

```js
var gulp = require('gulp');
var jshint = require('gulp-jshint');

gulp.task('test', function () {
  return gulp
    .src('./sample.js')
    .pipe(jshint())
    .pipe(jshint.reporter('jshint-stylish'));
});
```

You're done! That's all there is to declaring a Gulp task named `test`. It can be run using the command below, provided that you installed the `gulp` CLI globally.

```shell
gulp test
```

That was quite a trivial example. Just as well as ou can pipe the output of the JSHint linter through a reporter that will print the results of the linting test, you could also write output to disk by using `gulp.dest`, which creates a write stream. Let's step through another build task.

## Building a library in Gulp

To get started, let's do the bare minimum: read from disk with `gulp.src` and write back to disk piping the contents of the source file into `gulp.dest`, effectively just copying the file into another directory.

```js
var gulp = require('gulp');

gulp.task('build', function () {
  return gulp
    .src('./sample.js')
    .pipe(gulp.dest('./build'));
});
```

Copying the file is nice, but it doesn't quite minify its contents. To do that, you'll have to use a Gulp plugin. In this case you could use `gulp-uglify`, a plugin for the popular UglifyJS minifier.

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('build', function () {
  return gulp
    .src('./sample.js')
    .pipe(uglify())
    .pipe(gulp.dest('./build'));
});
```

As you've probably realized, streams enable you to add more plugins while only reading and writing to disk once. As an example, let's pipe through `gulp-size` as well, which will calculate the size of the contents that are in the buffer, and print that to the terminal. Note that if you add it before Uglify then you'd get the unminified size, and if you add it after, you'll get the minified size. You could also do both!

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var size = require('gulp-size');

gulp.task('build', function () {
  return gulp
    .src('./sample.js')
    .pipe(uglify())
    .pipe(size())
    .pipe(gulp.dest('./build'));
});
```

Just to reinforce the point on composability, being able to add or remove pipes as needed, let's add one last plugin. This time I'll use `gulp-header` to add some license information to the minified piece of code, such as the name, the package version, and the license type.

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');
var size = require('gulp-size');
var header = require('gulp-header');
var pkg = require('./package.json');
var info = '// <%= pkg.name %>@v<%= pkg.version %>, <%= pkg.license %>\n';

gulp.task('build', function () {
  return gulp
    .src('./sample.js')
    .pipe(uglify())
    .pipe(header(info, { pkg : pkg }))
    .pipe(size())
    .pipe(gulp.dest('./build'));
});
```

Just like in Grunt, in Gulp you can define flows by passing in an array of task names to `gulp.task`, instead of a function. The main difference between Grunt and Gulp in this regard is that Gulp will execute these dependencies asynchronously, while Grunt executes them synchronously.

```js
gulp.task('build', ['build-js', 'build-css']);
```

In Gulp, if you want to run tasks synchronously you'll have to declare a task as a dependency and then define your own task. All dependencies are executed before your task starts.

```js
gulp.task('build', ['dep'], function () {
  // here goes the task that depends on 'dep'
});
```

If you take anything away from this article, have that be this quote.

> **It doesn't matter which tool you use, as long as it allows you to compose the build flows you need in a way that doesn't make you work too hard for it.**

[0]: http://bevacqua.io/buildfirst "JavaScript Application Design: A Build First Approach"
[1]: http://gulpjs.com/ "The Streaming Task Runner"
[2]: http://gruntjs.com/ "The JavaScript Task Runner"
[3]: http://substack.net/task_automation_with_npm_run "Task automation with `npm run`"
[4]: https://www.npmjs.org/package/grunt-ec2 "grunt-ec2 on npm"
