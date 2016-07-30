# Introducing [Gulp][1]

You might want to kick things off by [going through the README][2], although I would reccomend you skim through the source code, as there aren't really a lot of surprises there, just some well thought-out, and concise code.

As you might've read while skimming over their documentation, installing Gulp is very reminiscent of what you had to do with Grunt.

- Have a `package.json`
- `npm install -g gulp`
- `npm install --save-dev gulp`
- Create a `gulpfile.js`
- `gulp`

At this point, the main difference with Grunt is that they didn't take their **CLI** out of core _(at least not yet)_, which might confuse you if you don't [understand global installs][3] in `npm`.

Before heading to a sample gulpfile, let's look at how the API looks like. There's five methods, and that's all you need. That's awesome, a concise API really helps keep the module focused on just what it's good at, processing build tasks.

- `.src(globs[, options])` takes a [glob][4] and returns an input stream
- `.dest(path)` takes a path and returns an output stream
- `.task(name[, deps], fn)` defines a task
- `.run(tasks...[, cb])` runs tasks
- `.watch(glob [, opts], cb)` watches the file system

I must say, that API is just awesome. Let's look at a simple task to compile Jade templates, taken from Gulp's documentation. Here, `jade()` and `minify()` are Gulp plugins, we'll get to those in a minute.

```js
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```

There isn't much boilerplate code involved, due to the fact that we're just writing code as opposed to putting together a configuration object.

In Grunt you'd need some boilerplate to get this thing going, such as loading `npm` modules with a rather weird `grunt.loadNpmTasks` method, creating a task alias that combines the required tasks, and configuring each of them to do what you need. One of the issues with Grunt which is solved by Gulp is that a single monolithic configuration object forces you to jump through hoops in order to achieve the results you want. If you have a workflow which copies a file and minifies something else, and another one which copies an unrelated file, the `copy` task configuration ends up with two completely unrelated `copy` operations, albeit under different targets.

Gulp does a good job of showing how code over configuration can help prevent such an scenario where configuration ends up being confusing and hard to digest.

## Savoring a `gulp`

Consider this short sample `gulpfile.js`, adapted from what's on the docs for Gulp.

```js
var gulp = require('gulp');
var uglify = require('gulp-uglify');

gulp.task('scripts', function() {
  // Minify and copy all JavaScript (except vendor scripts)
  gulp.src(['client/js/**/*.js', '!client/js/vendor/**'])
    .pipe(uglify())
    .pipe(gulp.dest('build/js'));

  // Copy vendor files
  gulp.src('client/js/vendor/**')
    .pipe(gulp.dest('build/js/vendor'));
});

// The default task (called when you run `gulp`)
gulp.task('default', function() {
  gulp.run('scripts');

  // Watch files and run tasks if they change
  gulp.watch('client/js/**', function(event) {
    gulp.run('scripts');
  });
});
```

Even if you don't know Node streams, this is pretty readable, right? I'd argue it's more readable than a `Gruntfile.js` which does the same things, because in this case we're simply following the code, and guessing what it does becomes much easier then. Take out comments stating the obvious, and you've got yourself a terse `gulpfile.js`.

The fact that Gulp provides a reasonable `.watch` implementation as part of their core API is also encouraging, as that's a key piece of functionality which gives a lot of value to a build system during development. Support for [asynchronous task development][5] feels much more integrated in Gulp than it does in Grunt, where targets really complicate matters when passing values to tasks.

Generally speaking, the API provided by Gulp makes more sense and is easier to use than that in Grunt. That's more or less the argument for Gulp. A **clean, concise, and awesome** API. Simple plugins which do **one thing very well**, and not whatever they feel like, as witnessed in many Grunt tasks. Not everything is pink roses for Gulp, though, and there are a few downsides to it as well.

## Dissecting `/Gr?u(nt|lp)/`

Let's see where the comparison between both task runners breaks down. Gulp is streams all the way down, almost as if you were shell scripting. That is, if you "get" [Node streams][6]. Otherwise, you're going to have a bad time.

[![streams.jpg][7]][8]

That being said, if you're a Node person, it's hard to ignore the audacity with which Gulp has you set up a build flow using code, rather than configuration, like Grunt does. This is, however, an undeniable drawback of Gulp. Some people will just never get streams. They might be PHP workers, or some other server-side voodoo like Ruby, or Python, and not be familiar at all with Node streams and buffers. They might know Common.JS, but that's as far as they'll ever get from their comfort zone. For those people, Gulp will never be a choice over Grunt.

> While Gulp is easier to read, Grunt is easier to write, and sometimes that's more valuable.

Gulp is oriented to do build stuff, and more specifically, things which _deal in files_. That's pretty much the bottom-line of their "In, Out, Watch" API. This is good, bad, and something else. It's good because it focuses on doing one thing. Builds. That's it, you have some inputs, and then you have some outputs, using some transforms which help shape them. It's bad because doing non-build stuff is harder with _such a precise API_, sending out [build notifications][9], or spinning up [Amazon EC2 instances][10] goes against what Gulp is designed to deliver.

Gulp is **extremely new** and we'll have to see _how its ecosystem evolves_, but I don't expect deployment Gulp tasks to gain significant adoption over what already exists in Grunt. I think that's a good thing, I don't believe Gulp could "beat" Grunt in CI and deployment circles, whereas I think it'll completely take over simpler workflows which don't involve much more than building client-side assets and pushing to Heroku.

## Gulp _won't sip out_ Grunt

There are a few reasons why I believe Gulp won't push Grunt to the brink of dehydration out in the desert. If anything, it'll bring more attention to it, by pushing the boundaries of what JavaScript task runners can do. First off, Gulp won't "beat" Grunt because it isn't anyone's goal for that to happen, certainly not that of industry leaders.

<blockquote class="twitter-tweet" lang="en"><p>Grunt vs. Gulp: both serve different needs &amp; can coexist happily. See: <a href="http://t.co/fQ5G8HDNwl">http://t.co/fQ5G8HDNwl</a> &amp;&amp; <a href="https://t.co/joUVeRqj8T">https://t.co/joUVeRqj8T</a>. We&#39;ll support both</p>&mdash; Addy Osmani (@addyosmani) <a href="https://twitter.com/addyosmani/statuses/420870024379637760">January 8, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

This might come as news, but it shouldn't come as a surprise. A lot of effort went into the current state of Grunt, and it wouldn't make a lot of sense laying waste to that by porting it all out to the latest hot chick in town, you have to make a choice. Stick to what you've got, or go out chasing the popular blonde of mystery.

Secondly, like I've mentioned earlier, **Gulp introduces a barrier of entry that doesn't exist in Grunt**, non-Noders will have a hard time dealing with streams, pipes, buffers, asynchronous JavaScript in general _(promises, callbacks, whatever)_, and I just don't see how it can strive amongst non-Noders looking for a front-end build system, considering those conditions.

Furthermore, Gulp **doesn't solve any new problems** really. The API is awesome and straightforward, but it does complicate non-build tasks, and Grunt has the upper hand in this one. It boasts [over 2000 plugins][11] registered on `npm`, against the [~200-ish][12] going for Gulp. That being said, it'd be interesting to see the ability to straight up run Grunt tasks in Gulp, but I don't think it would ever stick. I doubt using `/Gr?u(nt|lp)/` would make your life any easier, no matter what. If you need both, that's _probably a sign that you should just stick with Grunt_.

![regex.png][13]

There's also a _speed factor_ involved. I'll leave the merits of such speed gains for you to mull over. The important take-away here should be that there isn't a one-size-fits-all answer. Gulp might be faster, Grunt might be more "all-encompassing", but at the end of the day, you'll have to choose one over the other. Don't use both in the same application. Don't be _that guy_.

<blockquote class="twitter-tweet" lang="en"><p>just switched a project from <a href="https://twitter.com/search?q=%23gruntjs&amp;src=hash">#gruntjs</a> to <a href="https://twitter.com/search?q=%23gulpjs&amp;src=hash">#gulpjs</a> - simpler code, &amp; build time on save during watch from 3-5sec to 10-20ms. I kid you not.</p>&mdash; Andy Joslin (@andytjoslin) <a href="https://twitter.com/andytjoslin/statuses/416424985662468097">December 27, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Something else we might need to factor in is the case of _Grunt not really grunting_ all that much these days. This is a worrysome factor you should also be taking into account.

## The boar is becoming kind of stale

Grunt might _drown on its own_. It sat on `0.4.1` for ages, before moving to an unimpressive `0.4.2` release, and it doesn't seem to be going places now, either. Activity on the [@gruntjs][14] Twitter account is kind of flat-lining these days, and that's not a good sign, either.

I'm really hoping this is just transitional as planning for `0.5.0` is underway, but I feel like the team moved on to other projects. While I wouldn't consider it abandoned, it's a concern that I haven't seen raised yet. What I'd love to see is an eventual `1.0.0` release with a re-imagined configuration structure that deals with _the problems we've experimented thus far_. Easier plugin loading, a watch mechanism in core similar to what Gulp did, simpler file description semantics, and a reduced overhead for configuring tasks in general.

Of course, it's easy to _want_ those things, but it's hard to implement them without breaking most of the existing 2000 plugins. Considering the plugin ecosystem is one of Grunt's most valuable assets, it'll be hard to get right a release plan that's both sensible and meaningful. We'll just have to sit and wait, or you might want to [go ahead and propose something][17] to be implemented in `0.5.0`.

## The case for doing nothing

Right wing UNIX extremists have [time][18] and [again][19] suggested doing nothing. Forget about Gulp, Grunt, whatever. Just do nothing. I don't agree with this sort of extremism, you might just be more comfortable writing everything in JavaScript. It does, however, hold some merit in its premise. In the case of Gulp, I do consider the `npm run` approach as a valid questioning of its purpose. 

Gulp is pretty close to doing "nothing", a la `npm run`, while at the same time it kind of does "something", like Grunt does. I think Gulp provides value in providing Windows support, but it does introduce a certain amount of complexity, so it's really a trade-off. You need to ask yourself what you're looking for. If it's just the simplicity, you might be better off just using `npm run`!

The case for Windows support might not hold a lot of meaning within the Node community itself, since most of us seem to be working on _*nix_, but it does become a factor in other communities, which Grunt [seems to be penetrating][20]. I agree you should use some flavor of `bash` for Windows, it's still a pain doing just about anything in the command-line, and there isn't really much to say in favor of _not using_ Grunt on Windows.

[![nothing.png][15]][16]

So use Gulp, use Grunt, _whatever_.

## Whatever, _But_

Grunt wins at teaching people how to do builds, and even then, it's pretty hard to put it [in terms anyone can understand][21], but it fails at keeping it short. Gulp wins at being terse and having a gorgeous API, but it fails at the entry level, because of streams being hard to grasp at first. In the low-risk low-gain corner we have `npm run`. It wins at not doing anything, resulting in no overhead, but it fails at being cross-platform, if that's something that worries you.

Make a choice by yourself, don't just pick something _because XYZ said so_. Pick the tool which works for you. The one you understand, are comfortable with. Above all, the one **that fits your needs**. Don't go blindly chasing the latest fad because someone else tells you to. Similarly, don't get stuck with monolithic jQuery applications _(just to give out an example)_, try something else. Innovate. Be the change you want to see in the world.

> **Be the change you want to see in the world.**

I need a drink.

  [1]: http://gulpjs.com/ "Gulp Streaming Build System"
  [2]: https://github.com/gulpjs/gulp/ "gulpjs/gulp on GitHub"
  [3]: http://blog.nodejs.org/2011/03/23/npm-1-0-global-vs-local-installation/ "npm 1.0: Global vs Local installation"
  [4]: https://github.com/isaacs/node-glob "isaacs/node-glob on GitHub"
  [5]: https://github.com/gulpjs/gulp#async-task-support "Asynchronous Task Support in Gulp"
  [6]: http://www.youtube.com/watch?v=lQAV3bPOYHo "Harnessing the awesome power of streams"
  [7]: https://i.imgur.com/pnEIqGO.jpg
  [8]: http://www.youtube.com/watch?v=lQAV3bPOYHo "Harnessing the awesome power of streams"
  [9]: https://npmjs.org/package/grunt-winston "grunt-winston can log data over different transports"
  [10]: https://github.com/bevacqua/grunt-ec2 "grunt-ec2 creates, deploys to, and shuts down Amazon EC2 instances"
  [11]: http://gruntjs.com/plugins "Grunt Plugin Listing"
  [12]: http://gratimax.github.io/search-gulp-plugins/ "Search for Gulp Plugins"
  [13]: https://i.imgur.com/Ih0Y1Zw.png "To generate #1 albums, 'jay --help' recommends the -z flag."
  [14]: https://twitter.com/gruntjs "@gruntjs on Twitter"
  [15]: https://img.youtube.com/vi/EQnaRtNMGMI/0.jpg
  [16]: http://www.youtube.com/watch?v=EQnaRtNMGMI "The Nothing Pitch"
  [17]: https://github.com/gruntjs/grunt/issues?milestone=7 "Grunt issues for the 0.5.0 milestone"
  [18]: http://substack.net/task_automation_with_npm_run "task automation with npm run"
  [19]: https://gist.github.com/substack/8313379 "introducing ./task.js, THE new javascript task runner automation framework"
  [20]: https://npmjs.org/search?q=grunt-php "although I've no factual information to support that claim"
  [21]: http://24ways.org/2013/grunt-is-not-weird-and-hard/ "Grunt for People Who Think Things Like Grunt are Weird and Hard"
