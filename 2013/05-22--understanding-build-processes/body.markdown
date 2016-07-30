# Why a build process? #

I figure a good idea, assuming you've never done this before, is start by laying out the benefits of using a well-defined build process.

Chances are, you've worked in a project where you had to take several steps before you could get your local environment to work. Such a setup process might look similar to this:

- Create the database by hand. Lucky you, it's just [three easy steps](http://www.linux.org/article/view/create-mysql-database-via-command-line "Create MySQL database via command line")!
- Restore a data dump into the database. `mysql -u root -p suicide_db < ./dump.sql`
- Maybe edit the [hosts](https://en.wikipedia.org/wiki/Hosts_(file) "hosts file explained") file _by hand_
- Configure local variables such as database authentication, listener port, etc. Usually done _by hand_
- Download the latest version of the code. `git pull`
- Download external libraries from a repository such as [npm](https://npmjs.org/ "npm packages"), [NuGet](http://nuget.org/ "NuGet repository"), [gem](http://rubygems.org/ "Ruby gems"), [pip](https://pypi.python.org/pypi/pip "Python package index"). `npm install`
- Compile your code (in non-interpreted, compiled languages). `msbuild`
- Upgrade the database to the latest version. Generally done by a _post-build event_
- Run every unit test. `java junit.swingui.TestRunner test.Run`
- Compile assets such as [CoffeeScript](http://coffeescript.org/ "CoffeeScript Language") and [SASS](http://sass-lang.com/ "SASS Language"). `coffee --compile --output ./js ./bin`
- Start the web server. `node ./server.js`

Granted, this is _just for the first time_, or so you tell yourself. But the truth is, except for a few steps such as restoring the database, you are going to _run every single of these steps in **production**_. Well, actually, you are going to want to add a few more steps for a production build. Lets see:

- Start fresh, **don't leave anything behind**, except the _data_
- Compile your code in _release_ mode, removing debugging symbols
- Bundle and minify assets. `uglifyjs ./foo/main.js -o ./bin/foo.min.js`
- Finally, point the web server to the new version

Sure, frosting on our cake. But you can't deny **it adds up**. Lets see, **oh!** We might also want to do crafty things in our local environment.

- Display [full stack traces](/2013/03/06/defensive-design "Defensive Design") when things blow up
- Use the _unminified_ versions of vendor libraries such as [jQuery](http://jquery.com/ "jQuery library")
- Execute all the build steps again after making any changes

And think of the benefits of a one step build process! 

You could now hook your repository with continuous integration platforms that **alert you** in case your tests are _acting up_, and from there, think just how easy it becomes to `push` changes to different environments. 

> The question should then be: **why _not_ use a build process?**

# Where to start? #

The very first thing you need to do, is **investigate**. Learn about the _different build tools_ out there and find one that fits your project. It's usually a safe bet to pick the most popular build tool around your language. Once you've picked a tool, you'll need to **identify the core steps** it takes to build and run the application, and how those steps _mutate for each environment_. Find _commonalities_.

After you've identified these steps, you will have to decide which of those steps will pertain to the _build process_. You might entertain the idea of doing **absolutely everything** in that one step, and it might be the right thing to do. But maybe you can leave aside the database creation, while keeping the upgrades, for example. At this point you'll have to decide whether you want the build step to also function as a **"first-time setup"**, or not.

# Build step by step, in depth #

Lets go back to the _setup process_ I outlined earlier, and examine those steps.

> - Create the database by hand. Lucky you, it's just [three easy steps](http://www.linux.org/article/view/create-mysql-database-via-command-line "Create MySQL database via command line")!
> - Restore a data dump into the database. `mysql -u root -p suicide_db < ./dump.sql`

I'd leave steps like these to **a separate flow**, designed to set up the local development environment. The reason is _obvious enough_. We don't want to be _dropping the database_ and filling it with fake data at any point other than when we set up our local environment. These are probably the only couple of steps I'd be comfortable doing manually, and by that I mean: not in a single command execution. 

> - Maybe edit the [hosts](https://en.wikipedia.org/wiki/Hosts_(file) "hosts file explained") file _by hand_
> - Configure local variables such as database authentication, listener port, etc. Usually done _by hand_

This kind of configuration usually changes over time, so you must have good, **up-to-date** documentation on how to set up these things. The best thing you can do about this is either add part of this to the _setup flow_ we discussed earlier, or alternatively, add default configuration files that are _obvious enough_ to let developers get set up on their own.

> - Download the latest version of the code. `git pull`
> - Download external libraries from a repository such as [npm](https://npmjs.org/ "npm packages"), [NuGet](http://nuget.org/ "NuGet repository"), [gem](http://rubygems.org/ "Ruby gems"), [pip](https://pypi.python.org/pypi/pip "Python package index"). `npm install`

In the local environment, it might be sensible pulling code and installing dependencies by hand, rather than in an automated way, so that you have finer control over these things. Outside the local environment, this kind of steps will depend on the platform you are using.

In the case of this blog, for example, I am using [Heroku](https://www.heroku.com/ "Heroku Cloud Application Platform"). Heroku takes care of both of these steps. Well, I `git push heroku master`, and they respond by pulling that code, executing `npm install`, and running the application with a command I provide. [Travis](https://travis-ci.org/ "Travis: Free Hosted Continuous Integration") provides a free CI service, which also takes care of fetching my latest changes and installing dependencies. This might not always be the case, so you should learn your integration or hosting platform, maybe you _do have_ to push your dependency packages as well.

> - Compile your code (in non-interpreted, compiled languages). `msbuild`
> - Upgrade the database to the latest version. Generally done by a _post-build event_

In the case of the blog, I don't really have to compile anything, and [Mongoose](http://mongoosejs.com/ "Mongoose ODM") takes care of keeping my schemas up to date, [MongoDB](http://www.mongodb.org/ "MongoDB database engine") doesn't really care at all. But, _generally speaking_, this step will entail compiling your code using the command-line version of your language's compiler, and somehow updating the database. [DbUp](https://code.google.com/p/dbup/ "Upgrading SQL server databases the right way") is probably an awesome example of how this is done in **.NET**.

> - Run unit tests. `java junit.swingui.TestRunner test.Run`

Your code compiled, that's your first line of defense. In the case of JavaScript, that'd be a [linter](http://www.jshint.com/ "JSHint Code Quality Tool"). Now your test suite needs to give the green-light. Build tools are usually very well-suited to execute unit test runs, and output the test logs, so this one shouldn't be an issue once you learn the quirks of configuring the tool to run the tests.

> - Compile assets such as [CoffeeScript](http://coffeescript.org/ "CoffeeScript Language") and [SASS](http://sass-lang.com/ "SASS Language"). `coffee --compile --output ./js ./bin`
> - Bundle and minify assets. `uglifyjs ./foo/main.js -o ./bin/foo.min.js`

Nowadays, it is very common in web applications to resort to some little [DSL](http://en.wikipedia.org/wiki/Domain-specific_language "Domain Specific Language"), in order to avoid redundancies such as writing all the prefixes to `border-radius`, a tool can help you with this. But you then need to compile it, or you need an asset manager such as [assetify](https://github.com/bevacqua/node-assetify "assetify: the Node asset manager") or [SquishIt](https://github.com/jetheredge/SquishIt "asset optimization library for .NET"), to do that for you.

Sometimes those very libraries can handle bundling and minification as well, it's usually best to do it with the least amount of libraries, that translates as _less compatibility issues_ and **less headaches**.

> - Start the web server. `node ./server.js`

This too is _platform dependent_. Maybe you just want to do this locally for convenience. Maybe you don't need to do it _yourself_ in your hosting platform, but _someone_ has to trigger the web server to listen for incoming requests.

> - Execute all the build steps again after making any changes

This might currently be one of the most sought after, _and trending_, build steps. But in order to have a build process that restarts itself, using something like [grunt-watch](https://github.com/gruntjs/grunt-contrib-watch "grunt-contrib-watch on GitHub"), you must first have a very solid build process that deals with virtually everything I've discussed so far.

I've tried [WebStorm](http://www.jetbrains.com/webstorm/ "WebStorm JavaScript IDE") but I don't think I'm sold on the whole [live-edit](http://blog.jetbrains.com/webide/2012/08/liveedit-plugin-features-in-detail/ "LiveEdit plugin features in detail") thing, mainly because _it doesn't play very well_ with others. Meaning that if you have a custom build process, chances are it's going to _break down_.

I'm fine with _tabbing away_ from [my editor](http://www.sublimetext.com/ "Sublime Text Editor"), and just _refreshing_ my browser. The trick is to enable auto-save. `"save_on_focus_lost": true`

You can check out the [ponyfoo](https://github.com/bevacqua/ponyfoo "ponyfoo platform repository") repo if you want to figure out how I've configured the build process for this site.

# A Book? #

I'm currently engaged in talks regarding **writing a book** on the subject of _build processes_, maintainable software _architecture_, and in particular, how these concepts are _applied in JavaScript_, a language where these things used to be **utterly disregarded**, and are now beginning to get some affection.

> Your feedback and suggestions are more than welcome!
